.. SPDX-License-Identifier: GPL-2.0+

Omarchy Silicon release and boot-slot design
============================================

Status
------

This is the B-03/B-04 design note for ``omarchy-silicon/u-boot-omarchy``. It
is a proposal for implementation and qualification. It is not an implementation,
a release approval, a hardware qualification record, or a recovery claim.

This lane is intentionally limited to this document. The canonical program is
``omarchy-apple-platform/PROGRAM.md``. That program makes
``platform-manifest/v1`` the authority for the release tuple and
``boot-health/v1`` the authority for the boot-health record. This document
describes the U-Boot binding to those contracts; it does not create a second
schema authority.

The coordinator has fenced ``m1n1-omarchy`` as an opaque human-produced
artifact boundary because that repository contains an instruction prohibiting
AI/LLM use. This document does not inspect, read, analyze, edit, test, clone,
or make claims about that repository or its contents. The m1n1/DT tuple is
therefore an input to the experiments below, supplied and attested by the
human owner of that boundary.

Decision summary
----------------

The recommended design is:

* U-Boot is the slot authority after the preceding Apple boot handoff. It
  selects a slot only after validating the control record, the exact board
  identity, the signed per-slot manifest, and every artifact digest needed for
  the selected boot path.
* The Apple EFI System Partition remains the stable outer handoff surface.
  Each Omarchy slot has a disjoint, complete payload directory on that ESP.
  A small immutable or separately provisioned recovery payload is outside the
  normal A/B trial path.
* A pair of Omarchy-owned GPT control partitions stores redundant fixed-size
  boot-control records. The records contain a sequence number, slot state,
  selected and last-known-good slots, attempt information, the manifest
  digest, and a CRC. The platform manifest owns the final field definitions.
* The inactive slot is populated and verified before the control record is
  advanced. A power loss can leave a stale control copy, a torn new copy, or
  an incomplete inactive slot, but must never make an incomplete slot the
  selected slot.
* U-Boot writes the pending-attempt record before launching the selected
  payload. Linux writes the success or explicit failure result directly to the
  control store after the boot-health contract passes. The transport to Linux
  is a versioned ``/chosen`` device-tree handoff, with a kernel command-line
  fallback only as a diagnostic aid, never as the authority.
* GRUB is a payload and optional user interface inside the selected slot. It
  does not own slot state, consult ``BootNext``/``BootOrder`` for selection, or
  choose an unvalidated cross-slot kernel. U-Boot selects the slot and launches
  only that slot's validated GRUB and configuration.
* UEFI variables are not part of the durable update or boot-health protocol.
  In particular, the design does not depend on UEFI runtime
  ``SetVariable()``. U-Boot's file-backed variable mode and volatile runtime
  variable mode are separate behaviors and must not be confused with an Apple
  firmware variable service.

Existing evidence in this tree
------------------------------

The following are observed mechanisms in the checked-out U-Boot tree. They are
evidence for feasibility of individual pieces, not evidence that the complete
Omarchy design already works.

Apple board and image boundary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``doc/board/apple/m1.rst`` documents an AArch64 Apple build using
``apple_m1_defconfig`` and a separate ``u-boot-nodtb.bin`` output. It also
documents the board-specific payload assembly and the Apple EFI installation
handoff. Those instructions establish a reproducible-build starting point,
not a reproducibility result for an Omarchy release tuple.

The same board document lists the currently documented M1/M2 SoC families and
their limited U-Boot hardware support. It must not be read as the Omarchy
support registry, and it does not promote any board to FULL compatibility.
The exact board, SoC, firmware schema, and capability status come from the
signed platform manifest and the qualification ledger.

The Apple board code takes its working device tree from an address supplied by
the preceding boot stage, selects memory mappings from the root compatible
string, and fails on an unsupported SoC. It locates an Apple NVMe EFI System
Partition using GPT EFI attributes, preferring the partition UUID supplied in
the working device tree when present. The Apple environment integration also
uses that ESP for FAT-backed U-Boot environment storage. These are useful
interfaces, but the mutable U-Boot environment is not a safe slot authority:
it is not the redundant, digest-bound transaction record proposed here.

UEFI and boot-standard mechanisms
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The generic UEFI loader can execute an AArch64 EFI application and can pass a
device tree to it. The generic EFI boot manager uses ``BootXXXX``,
``BootNext``, and ``BootOrder`` variables, while standard boot can discover an
EFI file from a boot flow. The Apple release profile must use explicit slot
paths and a validated manifest rather than allowing generic discovery or EFI
boot-manager ordering to select a different slot.

The generic UEFI configuration has three materially different variable
behaviors:

* ``EFI_VARIABLE_FILE_STORE`` persists variables in ``/ubootefi.var`` on the
  ESP, but this file is not a runtime write channel for the operating system.
* ``EFI_RT_VOLATILE_STORE`` exposes runtime variable writes backed by RAM; the
  operating system would have to synchronize those writes to a file, and they
  are not durable by themselves.
* ``EFI_VARIABLE_NO_STORE`` does not persist non-volatile variables. Runtime
  ``SetVariable()`` is unsupported unless the volatile runtime store is
  enabled, even though U-Boot may still expose other EFI runtime entry points.

The release profile must pin the intended choice explicitly in its resolved
configuration and test the resulting EFI table. It must not infer the choice
from a Kconfig default or treat the presence of ``GetVariable()`` as proof of
durable variable storage.

GPT and update mechanisms
~~~~~~~~~~~~~~~~~~~~~~~~~

U-Boot's GPT support provides stable disk and partition GUIDs, CRC-protected
primary and backup GPT headers, ``gpt verify``, and partition lookup by name,
type, or unique GUID. The GPT tests exercise writing, reading, type GUIDs,
bootable attributes, and transposition on sandbox media. Those facilities are
appropriate for resolving the Omarchy control partitions and validating the
installation plan, provided every identifier is revalidated before mutation.

U-Boot also contains a generic FWU multi-bank implementation. Its metadata has
two copies, active and previous bank indexes, bank states, image GUIDs, and
CRC validation. The FWU documentation and sandbox DM tests demonstrate a
possible foundation for redundant metadata and image identity.

The current generic FWU trial path is not yet the Apple design: its trial
counter is an EFI variable, and its platform boot-index hook is supplied by an
earlier boot stage or falls back to the active index. Neither behavior is a
proven Apple boot-health transport under the coordinator's m1n1 boundary. FWU
reuse is consequently an implementation experiment, not an approval to turn
on the existing options unchanged.

Bootcount and bootflow
~~~~~~~~~~~~~~~~~~~~~~

The generic bootcount feature increments before autoboot and dispatches to
``altbootcmd`` after ``bootlimit``. It is useful as a conceptual fallback
guard, but its environment-backed behavior does not provide the manifest-bound
two-copy record, selected-slot identity, or OS success proof required here.
The Apple implementation should use the proposed control-store state machine
and may adapt the generic bootcount code only after its persistence and power-
loss semantics are proven.

Standard boot's bootflow framework tries boot devices, methods, and flows in
order. That behavior is valuable for sandbox tests and diagnostics. It is too
permissive as the release selector if an unqualified extlinux file, arbitrary
EFI file, USB device, or stale environment can win over the selected Omarchy
slot. Release autoboot must enter through the slot state machine first.

B-03: reproducible Apple configuration and release contract
------------------------------------------------------------

Configuration ownership
~~~~~~~~~~~~~~~~~~~~~~~

The release pipeline shall resolve one explicit U-Boot configuration per
qualified Apple board/configuration class. A generic ``apple_m1_defconfig``
may remain the bring-up baseline, but a release must record the complete
resolved ``.config`` rather than relying on default-y behavior. In particular,
the release configuration must make an intentional decision for:

* AArch64, device-tree control, Apple NVMe, FAT read/write requirements, and
  the exact console/input path needed for unattended boot and manual recovery.
* EFI binary execution, EFI device-path behavior, and whether generic EFI boot
  manager support is excluded from release selection.
* Hash/signature verification for the platform manifest and every loaded
  payload. EFI secure boot, Apple owner authorization, and LocalPolicy are
  separate trust boundaries; enabling one must not be reported as the others.
* The durable variable choice. The recommended first experiment is no durable
  EFI-variable dependency, no volatile runtime variable store, and direct
  selection of the signed slot path. If file-backed variables are retained for
  diagnostics or compatibility, they remain non-authoritative and are tested
  as stale, missing, corrupt, and unwritable inputs.
* The boot-control block reader/writer, GPT partition support, recovery path,
  and the minimum console/menu support needed to recover without a valid
  mutable environment.

The final config record must include the source commit, Kconfig fragment or
defconfig inputs, resolved config hash, compiler and binutils identities,
device-tree compiler identity, host/build container digest, and the exact
manifest schema version. A config that happens to compile on a developer host
is not a release record.

Reproducible build procedure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For each board tuple, the release builder shall:

* start from a clean source checkout at the manifest-pinned U-Boot commit;
* use an allowlisted, digest-pinned AArch64 toolchain and declared host
  container;
* resolve the configuration in an out-of-tree build directory;
* set ``SOURCE_DATE_EPOCH`` to the manifest's release timestamp and keep
  locale, timezone, and tool versions declared;
* build U-Boot, its generated device trees, and the release payload bundle;
* record SHA-256 digests for the source, resolved configuration, U-Boot binary,
  each DTB, each signed manifest, and each slot artifact; and
* repeat the build independently in a fresh output directory and compare the
  complete artifact set byte-for-byte. A difference requires an identified,
  reviewed source of nondeterminism or the candidate is rejected.

The existing U-Boot reproducible-build document proves that
``SOURCE_DATE_EPOCH`` controls build timestamps. It does not prove that the
Apple image assembly, signing, device-tree generation, or outer Apple handoff
are reproducible. Those must be included in the independent-build comparison.

Platform manifest handoff
~~~~~~~~~~~~~~~~~~~~~~~~~

``platform-manifest/v1`` remains the only release-tuple authority. U-Boot
consumes a generated binding and must reject an absent, unsigned, expired,
malformed, or board-mismatched manifest before selecting a slot. The U-Boot
binding must not copy the canonical schema into a second hand-maintained field
list.

Conceptually, each slot contains a complete manifest-bound bundle like this::

    EFI/OMARCHY/slots/A/manifest
    EFI/OMARCHY/slots/A/manifest.sig
    EFI/OMARCHY/slots/A/grubaa64.efi
    EFI/OMARCHY/slots/A/grub.cfg
    EFI/OMARCHY/slots/A/Image
    EFI/OMARCHY/slots/A/initramfs.img
    EFI/OMARCHY/slots/A/<board-specific-dtb>

The same layout exists for slot ``B``. The names are illustrative until the
platform manifest defines artifact roles. A manifest digest is bound to the
slot record, and every file loaded by the chosen path is checked against the
manifest before execution. A valid signature on GRUB alone is insufficient if
GRUB can load an unrelated kernel, initramfs, or DTB.

The manifest handoff must carry, directly or through generated bindings, at
least the exact board identity, SoC identity, firmware schema, source and
artifact digests, DT compatibility allowlist, root storage identity, slot
identity, rollback compatibility, and signing key identity. Unknown fields or
schema revisions fail closed until the consumer binding is updated.

The preceding Apple boot stage may supply a DT pointer and other boot context,
but a transient register, environment variable, or unverified boot argument
must not override the durable slot record. Any preceding-stage slot hint is
only a cross-check and diagnostic input until the human owner provides and
qualifies the actual handoff contract.

UEFI release behavior
~~~~~~~~~~~~~~~~~~~~~

The release boot path is intentionally direct:

1. Resolve the Omarchy-owned NVMe disk and Apple ESP using stable GPT identity
   and the board's device-tree context.
2. Read and validate the redundant control records.
3. Resolve the selected slot's manifest and signature from the fixed slot path.
4. Verify board, firmware schema, slot, root identity, and artifact digests.
5. Load and execute only the selected slot's validated GRUB or direct-kernel
   payload, passing the boot-health handoff.

Generic ``BootOrder``/``BootNext`` selection, a search through every ESP, and
an arbitrary ``bootaa64.efi`` fallback are not release behavior. They may be
exposed in a developer build for diagnostics, but the release menu must label
them as manual and unqualified paths.

The Apple ESP preference already present in this tree is useful for locating
the stable outer partition. It does not solve slot selection, artifact
authentication, or transaction atomicity. The implementation must also test
multiple OS-owned ESPs and an ``asahi,efi-system-partition`` UUID that is
missing, stale, duplicated, malformed, or points at a partition with the
wrong GPT type.

EFI variable runtime boundary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are two distinct questions:

* Does U-Boot expose an EFI runtime-services table to an EFI application or
  kernel?
* Is there a durable, shared, power-loss-safe variable store that Linux, GRUB,
  U-Boot, and the Apple outer firmware all update consistently?

This design requires neither question to be answered in the affirmative for
slot state. The release must not use ``BootNext``, ``BootOrder``,
``OsIndications``, ``TrialStateCtr``, or an EFI ``SetVariable()`` call as the
boot-success or fallback protocol. The current generic FWU code's use of
``TrialStateCtr`` is therefore an explicit integration gap.

The first UEFI experiment shall boot a small probe through the exact Apple
release path and record the status of ``GetVariable()``, ``SetVariable()``,
``QueryVariableInfo()``, and ``UpdateCapsule()`` before and after
``ExitBootServices()``. The test must also reboot after each write and verify
whether anything persisted. The expected release behavior is that a failed or
unsupported EFI-variable operation leaves the control-store state unchanged
and does not block manual recovery.

B-04: versioned slots, attempt state, and fallback
--------------------------------------------------

Storage layout
~~~~~~~~~~~~~~

The installer and platform manifest must provision the following Omarchy-owned
objects without modifying unrelated Apple or macOS containers:

* one stable Apple ESP containing the outer handoff and the disjoint ``A``,
  ``B``, and recovery payload paths;
* two small, separately addressable GPT control partitions containing one
  fixed-size record copy each; and
* the versioned root-storage identity required by each slot, as described by
  the canonical installer and platform manifests.

The control partitions use a platform-assigned GUID/type from the canonical
registry. U-Boot must resolve them by disk GUID, partition type/unique GUID,
and expected size, then re-read those identifiers immediately before a write.
It must never accept a raw device path, glob, mutable environment value, or
unresolved variable as the destructive target.

Each slot directory is independently complete. Updates write temporary names
inside the inactive slot, flush and verify each artifact, verify the signed
manifest last, and leave the current slot untouched until the control record
switch is committed. A partially written inactive slot is invalid because its
manifest is absent, unsigned, mismatched, or has a failed artifact digest.

The conceptual control record is:

* format version and magic;
* monotonically increasing sequence number;
* disk GUID and control-partition-set identity;
* selected slot, last-known-good slot, and optional recovery state;
* stable, trial, failed, or recovery state;
* attempt budget and attempts consumed for the selected trial;
* selected manifest digest and release identifier;
* last failure class and bounded diagnostic data; and
* CRC over the record, with reserved bytes required to be zero.

The canonical ``boot-health/v1`` schema defines the cross-repository meaning
of these values. The on-disk encoding, endian rules, record size, and GUIDs
must be generated or imported from that authority before implementation.

Record selection and write ordering
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On every boot U-Boot reads both control partitions, validates magic, version,
CRC, disk identity, sequence monotonicity, state ranges, slot names, and the
manifest digest shape. It selects the highest-sequence valid record. If only
one copy is valid, it may repair the other copy after the selected boot state
is safe; it must not silently invent a new state.

To commit a new state:

1. Read both records and choose the next sequence number.
2. Populate a complete new record in memory and calculate its CRC.
3. Write one control copy, flush it, read it back, and compare it byte-for-byte.
4. Write the other copy, flush it, and read it back.
5. Only after the durable record is valid may U-Boot launch the selected slot.

A write failure or torn record leaves the other valid copy authoritative. If
both copies are invalid or disagree in a way that cannot be resolved by
sequence and CRC, U-Boot must stop at recovery/manual selection and report the
record failure. It must not guess from a directory name or generic bootflow.

A complete update transaction is:

* verify the signed platform manifest and board tuple before mutation;
* select the inactive slot from the last-known-good record;
* populate the inactive slot without changing the active slot;
* verify all artifacts, the manifest digest, the root identity, and the slot's
  GRUB-to-kernel references;
* commit ``trial`` with the inactive slot, manifest digest, and bounded attempt
  budget; and
* reboot once through the same selection path.

The active slot is never overwritten in place. A platform that cannot provide
that invariant for a particular artifact must keep that artifact outside the
slot transaction and explicitly require a separate, experimentally qualified
update protocol.

Attempt and success transport
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before launching a trial, U-Boot consumes one attempt in a new durable control
record. This makes an unclean reset, power loss, kernel panic, GRUB hang, or
process death observable on the next boot without relying on EFI variables.
The attempt is associated with the exact slot and manifest digest, not merely
with a global integer.

U-Boot then adds a generated ``boot-health/v1`` handoff under ``/chosen`` in
the final device tree. The binding must include the selected slot, attempt
number, attempts remaining, manifest digest, control-partition identity, and
the supported success-writer version. The kernel-side health writer must
reject a success report whose slot, digest, board, or control identity does
not match the handoff.

The OS health writer is responsible for the success transition only after the
required board smoke contract passes. It reads the handoff, checks that the
running root and boot artifacts match it, and writes an ``accepted`` record to
both control copies using the platform-defined writer. It may also write an
explicit, typed failure record. A generic service-start message or a desktop
reaching a login screen is not automatically a boot-health success.

Preserving the ``/chosen`` handoff through GRUB is an experiment. If the exact
GRUB path does not preserve it, the release must add a versioned, authenticated
handoff mechanism and test it. A kernel command-line value may aid diagnosis,
but it is mutable input and cannot authorize a success write.

Fallback rules
~~~~~~~~~~~~~~

The selection state machine shall behave as follows:

* If the current record is stable, validate and boot its last-known-good slot.
* If the record is trial, validate the trial slot and consume one attempt
  before launching it. If validation fails, record a typed failure and use the
  last-known-good slot without selecting arbitrary media.
* If the trial reaches the attempt limit without an accepted health record,
  mark it failed, restore the last-known-good slot as selected, and retain the
  failed manifest digest for diagnosis.
* If an explicit OS failure arrives, mark the trial failed and fall back on the
  next reboot. If the failure writer cannot persist, the unaccepted trial
  remains bounded by the attempt limit.
* If the last-known-good slot is also invalid, select the signed recovery
  payload. If recovery is absent or invalid, stop with a diagnostic and expose
  manual recovery; do not claim that the machine is recovered.
* USB, network, or another ESP is never an automatic fallback in a release
  build. Such media can be selected manually only after its manifest, board,
  and artifact identities pass the same validation.

Manual recovery
~~~~~~~~~~~~~~~

Manual recovery must work with a corrupt or missing mutable U-Boot environment.
The immutable compiled defaults and the control-record reader must be enough
to show:

* both record copies and their validity/sequence numbers;
* selected, last-known-good, trial, failed, and recovery states;
* each slot's manifest digest, board match, artifact failures, and remaining
  attempts; and
* the exact reason an automatic choice was rejected.

The eventual command/menu names are implementation details, but the intended
operations are equivalent to ``status``, ``try slot A/B once``, ``select a
validated last-known-good slot``, ``boot signed recovery``, and ``halt``. A
manual force of an unsigned or board-mismatched artifact must not be exposed as
a normal recovery operation. The Apple boot picker, RecoveryOS, owner
authorization, and DFU path remain outer installer/recovery responsibilities;
this document does not claim any of them has been tested or recovered.

GRUB interaction
~~~~~~~~~~~~~~~~

GRUB is launched from the already selected slot and is not permitted to become
a second slot state machine. The required contract is:

* U-Boot selects ``A`` or ``B`` and verifies the slot manifest before loading
  ``grubaa64.efi``.
* GRUB reads only that slot's configuration and payload references for normal
  boot. It must not follow a mutable global ``grub.cfg`` to another slot.
* GRUB receives or preserves the selected slot and manifest identity for the
  Linux boot-health handoff.
* A GRUB load/verification/launch failure returns to U-Boot's bounded fallback
  path. A GRUB menu timeout is not a success marker.
* Linux failure after GRUB has transferred control is handled by the durable
  attempt budget and the absence of the OS success record.
* A human-selected recovery entry is clearly labeled and still uses a signed,
  board-matched recovery manifest.

The existing U-Boot EFI boot-manager and distro/bootflow tests can validate
generic EFI application execution and GRUB interaction in sandbox or QEMU
contexts. They do not validate Apple slot identity, a real GRUB handoff, or
the absence of an Apple EFI variable service. Those are separate experiments.

m1n1 and device-tree compatibility boundary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No source or behavior claim about ``m1n1-omarchy`` is made here. The U-Boot
side of the boundary must define and test only the inputs it consumes:

* a valid device-tree pointer and an exact root-compatible/board identity;
* the firmware schema and release tuple identity expected by the selected
  manifest;
* the Apple ESP identity and any working ``/chosen`` properties required to
  locate storage; and
* a documented memory and reserved-memory map that does not overlap loaded
  U-Boot, DT, GRUB, kernel, initramfs, or framebuffer regions.

The U-Boot implementation must reject a missing or malformed DT, an unknown
root compatible, a board/SoC mismatch, an unsupported firmware schema, and an
ESP identity mismatch before a slot is selected. It must preserve required
``/chosen`` data while adding the boot-health handoff. Device-tree changes are
an ABI change and require a new manifest compatibility declaration.

The human owner of the opaque predecessor boundary must supply the pinned
tuple, ABI expectations, and test fixture. The U-Boot lane can then run a
black-box handoff test and report pass/fail without inspecting that repository.

Proven mechanisms versus experiments
-------------------------------------

Proven feasible mechanisms in this tree include:

* Apple AArch64 U-Boot configuration and documented payload assembly;
* Apple working-DT consumption and exact SoC-compatible dispatch;
* Apple NVMe/ESP discovery using GPT EFI attributes and a selected ESP UUID;
* FAT-backed U-Boot environment access, with its mutable-state limitations;
* UEFI EFI-application execution and standard boot/bootflow infrastructure;
* GPT identity, CRC validation, type GUIDs, and sandbox GPT tests;
* generic two-copy FWU metadata, active/previous indexes, bank states, and
  image GUID lookup; and
* generic bootcount and alternate-boot concepts.

The following remain proposals requiring implementation and experiments:

* generated bindings for ``platform-manifest/v1`` and ``boot-health/v1``;
* signed per-slot manifest verification and digest-closed GRUB payload loading;
* the two Omarchy control partitions and sequence/CRC state machine;
* direct Apple NVMe block reads/writes for the control record, including
  flush/readback behavior;
* an OS success/failure writer that operates without EFI runtime variables;
* preservation and authentication of the boot-health handoff through GRUB;
* exact fallback behavior after a GRUB failure, kernel failure, reset, or
  power loss;
* the release Kconfig profile, including literal EFI runtime behavior; and
* compatibility with the human-supplied predecessor/DT tuple across every
  qualified board and firmware schema.

Acceptance and power-loss test plan
------------------------------------

Existing tests to reuse or extend
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The first implementation should extend the existing test surfaces rather than
claiming that generic tests cover the new contract:

* ``test/dm/fwu_mdata.c`` for redundant metadata read/write, then add record
  generation, CRC, copy divergence, and malformed-state cases;
* ``test/py/tests/test_gpt.py`` for stable GUID/type/size resolution and
  rejection of an ambiguous control-partition layout;
* ``test/boot/bootflow.c`` and the bootmeth tests for explicit slot bootflow
  ordering and rejection of unqualified global discovery;
* ``test/py/tests/test_efi_bootmgr.py`` for EFI application execution, while
  asserting that slot selection does not come from ``BootOrder``;
* ``test/py/tests/test_efi_capsule/`` for signed staging, rejected GUIDs,
  interrupted capsule handling, and removal of applied update files; and
* ``test/py/tests/test_distro.py`` as a reference for GRUB console interaction,
  not as Apple hardware or release evidence.

New sandbox tests should use a disposable host-bound disk image and inject
failures at each durable write boundary. The test harness must capture the
control copies, slot files, manifest digest, selected slot, attempt count, and
fallback decision after every simulated reset.

Required cases
~~~~~~~~~~~~~~

The B-04 gate is not complete until the following cases have deterministic
expected results:

* no valid control copy, one valid copy, divergent valid copies, a torn copy,
  stale sequence numbers, bad CRC, invalid state, and mismatched disk GUID;
* missing, unsigned, expired, schema-unknown, board-mismatched, and digest-
  mismatched manifests;
* missing or corrupt GRUB, configuration, kernel, initramfs, or DTB in the
  inactive slot, with the active slot remaining bootable;
* power loss before staging, after every artifact, after manifest verification,
  after control copy one, after control copy two, after attempt consumption,
  during GRUB, before Linux success, after success copy one, and after success
  copy two;
* repeated resets through a trial slot until the attempt budget is exhausted,
  followed by exactly one fallback to last-known-good and a durable failed
  state;
* an explicit OS failure and a success report with the wrong slot, digest,
  board, or control identity;
* a stale or corrupted ``ubootefi.var``, unsupported runtime ``SetVariable()``,
  volatile variable changes, missing ``BootNext``, and malicious or stale
  ``BootOrder``; none may change slot authority;
* GRUB selecting recovery, a GRUB return/error, a kernel that never emits
  success, and a recovery image that fails validation;
* GPT primary/backup corruption and a control partition disappearing between
  resolve and write; and
* black-box handoff fixtures for every qualified Apple board/firmware-schema
  tuple supplied by the human owner of the opaque predecessor boundary.

Power-loss evidence must include the injected cut point, durable bytes from
both control copies, the slot directory listing, the selected manifest digest,
the next boot decision, and whether repair was attempted. A green test that
only reaches a U-Boot prompt is not update atomicity evidence.

Qualification gates and open questions
---------------------------------------

The implementation gate is: exact resolved config, reproducible independent
builds, signed manifest and artifact verification, deterministic sandbox state
machine tests, EFI-variable boundary tests, GRUB handoff tests, and power-loss
tests all pass. The physical gate additionally requires destructive tests on
disposable qualified Apple hardware with a rehearsed outer recovery path. No
virtual, sandbox, compilation, or U-Boot prompt result can produce a physical
qualification record.

Questions that must be answered before implementation is accepted:

* What canonical platform-assigned GUIDs and generated bindings identify the
  two control partitions and the slot artifacts?
* Is the first release path direct kernel boot or GRUB, and which component
  verifies each artifact in that path?
* What exact device-tree property or protocol preserves ``boot-health/v1``
  through the selected GRUB version?
* How does the OS health writer obtain a safe, versioned write capability to
  the control partitions without EFI runtime variables?
* Which U-Boot implementation, if any, can provide durable record writes on
  Apple NVMe with flush and readback semantics, and what physical experiment
  proves it after power removal?
* What predecessor/DT ABI and firmware-schema evidence will the human owner
  provide at the opaque m1n1 boundary?
* What exact manual input path is guaranteed when the mutable environment,
  both A/B payloads, or the control copies are damaged?

Until these questions have experimental answers and the gates above pass,
B-03/B-04 remain DESIGN/BRINGUP work. Nothing in this note is DONE.
