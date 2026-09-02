.. SPDX-License-Identifier: GPL-2.0+

Omarchy Silicon release and boot-slot design
============================================

Status and scope
----------------

This is the B-03/B-04 design note for ``omarchy-silicon/u-boot-omarchy``, correction round 1. It is a design-only document. It is not an implementation, a build result, a schema, a binding, a test result, a compatibility claim, a support claim, a qualification record, a release approval, or a recovery claim. Nothing described here is DONE. B-03 and B-04 remain open, not-started slices in the canonical program ledger, and their implementation admission is BLOCKED until the upstream contracts named in this document are ratified.

The canonical program is ``omarchy-apple-platform/PROGRAM.md`` at the ratified commit ``58302d148f0e8b855578f9aa518ff1c5eb48c515``. This document binds U-Boot to the frozen cross-repository contracts listed in that program and creates no second authority.

Every wire detail derived from the F-02 candidate ``docs/design/platform-schema.md`` at ``c315c7e79928d0041deb582bed79a61074361b21`` is PROVISIONAL. That candidate is REJECTED and frozen pending the owner checkpoint recorded in the program. Provisional details are quoted so that the U-Boot binding is exact rather than illustrative; they are not a local shadow schema, they are not implemented here, and they change automatically to whatever the coordinator ratifies. Where an F-02-owned identifier is not ratified, this document uses a named BLOCKED constant from the table in `BLOCKED external constants`_ instead of a guessed value.

The coordinator has fenced ``m1n1-omarchy`` and every m1n1 path as an opaque human-produced boundary. This document does not inspect, read, analyze, edit, test, clone, fetch, browse, traverse, characterize, or make claims about that repository or its outputs. The predecessor stage appears here only as a coordinator-supplied signed opaque artifact envelope and as the observable inputs U-Boot consumes at its own entry point.

Verdict on the previous tip
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The coordinator rejected tip ``e385874ca1987248b40aa150f10d3d3ad32fde02`` with eight contract failures: unclosed cross-contract authority, a conceptual boot-control record, conflated success authority, an unclosed GRUB handoff, unenforced release selection, illustrative artifact roles, a missing toctree entry, and zero executable artifacts. Each is addressed in the section named below; none is addressed by implementation, because this lane owns only this document and ``doc/board/apple/index.rst``.

.. list-table::
   :header-rows: 1
   :widths: 6 44 50

   * - Failure
     - Ruling
     - Section
   * - 1
     - Bind exactly the ratified F-02/F-03 trusted seam
     - `Trusted seam binding`_
   * - 2
     - Fully specify the durable boot-control record and two-copy algorithm
     - `Durable boot-control record`_
   * - 3
     - Separate success authority from boot-control writing
     - `Authority separation`_ and `Success-mark transport`_
   * - 4
     - Close the m1n1 to U-Boot to GRUB release path and BootContext transport
     - `Release boot path`_
   * - 5
     - Make generic fallback structurally unavailable in release mode
     - `Release command and configuration profile`_
   * - 6
     - Close provenance and artifact roles with canonical manifest identities
     - `Provenance and artifact roles`_
   * - 7
     - Add the document to the Apple toctree
     - ``doc/board/apple/index.rst`` in the same change
   * - 8
     - Preserve the empirical NOT IMPLEMENTED census
     - `Empirical residuals`_

Existing evidence in this tree
------------------------------

The following facts were read from the checked-out U-Boot tree at base ``b33034a515a068381c4aa28831fb709e91300cd1``. They are evidence for the feasibility of individual mechanisms and for the accuracy of the release profile below. They are not evidence that any Omarchy contract exists or works.

* ``configs/apple_m1_defconfig`` sets ``CONFIG_BOOTCOMMAND="bootflow scan -b"``, ``CONFIG_USE_PREBOOT=y``, ``CONFIG_NO_NET=y``, ``CONFIG_NVME_APPLE=y``, and the USB xHCI, DWC3, keyboard, and framebuffer options. It does not set any Omarchy option because none exists.
* ``board/apple/mac/mac.env`` sets ``boot_targets=nvme usb`` and routes the console to ``vidconsole`` and ``serial`` with USB, SPI, and MTP keyboards on ``stdin``.
* ``arch/arm/mach-apple/board.c`` selects the memory map from the root ``compatible`` value for ``apple,t8103``, ``apple,t8112``, ``apple,t6000``, ``apple,t6001``, ``apple,t6002``, ``apple,t6020``, ``apple,t6021``, ``apple,t6022``, and ``apple,t8122`` and panics on any other SoC. ``asahi_esp_devpart()`` reads ``/chosen`` property ``asahi,efi-system-partition``, scans NVMe namespaces whose parent is ``apple,nvme-ans2``, prefers the partition whose UUID matches, and otherwise falls back to the first partition carrying the EFI system partition attribute. ``board_late_init()`` publishes ``fw_dev_part`` and the load addresses through the environment. ``ft_board_setup()`` rewrites ``/chosen/stdout-path`` when a keyboard and framebuffer are present.
* ``env/Kconfig`` defaults ``ENV_IS_IN_FAT`` to ``y`` for ``ARCH_APPLE``; the board supplies the ESP device and partition to that environment store. ``ENV_IS_NOWHERE``, ``ENV_WRITEABLE_LIST``, and ``ENV_ACCESS_IGNORE_FORCE`` exist as alternatives.
* ``lib/efi_loader/Kconfig`` defaults the non-volatile variable store to ``EFI_VARIABLE_FILE_STORE`` when ``FAT_WRITE`` is enabled, with the file ``/ubootefi.var`` on the ESP; ``EFI_VARIABLE_NO_STORE`` and ``EFI_RT_VOLATILE_STORE`` are the alternatives. ``EFI_BOOTMGR`` defaults to ``y``.
* ``boot/Kconfig`` defaults ``BOOTSTD`` to ``y`` when ``BLK`` is set, ``BOOTMETH_EFILOADER`` and ``BOOTMETH_EFI_BOOTMGR`` to ``y``, and ``BOOTCOMMAND`` to ``bootflow scan -lb`` under ``BOOTSTD_DEFAULTS``. ``boot/bootmeth_efi.c`` searches ``/EFI/BOOT/`` for ``bootaa64.efi`` on every scanned filesystem.
* ``lib/efi_loader/efi_helper.c`` ``efi_install_fdt()`` copies the device tree, runs ``image_setup_libfdt()`` (which applies ``ft_board_setup()`` and the ``/chosen`` fixups from ``boot/fdt_support.c``, including ``bootargs`` from the environment when set), carves out reservations, and installs the result as the ``EFI_FDT_GUID`` configuration table. ``include/efi_loader.h`` exports ``efi_binary_run(image, size, fdt, initrd, initrd_sz)``.
* ``include/part.h`` exports ``part_get_info_by_uuid()``, ``part_get_info_by_name()``, ``part_get_info_by_type()``, ``gpt_verify_headers()``, and ``gpt_verify_partitions()``; ``MAX_SEARCH_PARTITIONS`` is 128.
* ``lib/sha256.c`` provides ``sha256_hmac()``; ``lib/crc32c.c`` provides CRC-32C. There is no Ed25519 implementation and no JSON parser or RFC 8785 canonicalizer anywhere under ``lib/``.
* ``include/blk.h`` and ``drivers/block/blk-uclass.c`` expose no flush or barrier operation. ``drivers/nvme/nvme.h`` defines the ``nvme_cmd_flush`` opcode, and ``drivers/nvme/nvme.c`` never issues it; the only flushes in that driver are CPU data-cache flushes.
* ``include/fwu_mdata.h`` defines the generic two-copy FWU metadata with ``crc32``, ``version``, ``active_index``, ``previous_active_index``, and per-version bank or image state. The generic FWU trial counter is an EFI variable.
* ``test/dm/fwu_mdata.c``, ``test/py/tests/test_gpt.py``, ``test/boot/bootflow.c``, ``test/py/tests/test_efi_bootmgr.py``, ``test/py/tests/test_distro.py``, and ``test/py/tests/test_efi_capsule/`` exist. No Omarchy test exists.
* The host used for this correction has GNU Make 3.81, which rejects ``Makefile`` at line 247 with a missing separator. Sphinx and docutils are not installed. No configuration, build, or documentation build ran.

Trusted seam binding
--------------------

U-Boot binds to exactly the eight authenticated payload types frozen by the program and detailed by the F-02 candidate. It never accepts a ninth type, a consumer-local field list, or a role table of its own.

Envelope and verification order
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PROVISIONAL, pending coordinator approval. Every authenticated object is carried by the closed ``omarchy-signed/v1`` envelope with exactly ``format``, ``payload_type``, ``payload_version``, ``domain``, ``context``, ``schema_set_digest``, ``payload``, and ``signatures``. The payload carries the common fields ``schema``, ``schema_set_digest``, ``document_id``, ``issuer``, ``issued_at``, and ``expires_at``. Each signature entry carries exactly ``key_id``, ``signer_role``, ``algorithm = ed25519``, ``signature_format = raw-ed25519/v1``, and a 64-byte signature in unpadded base64url. ``payload_digest = sha256(JCS(payload))`` and the signed preimage is ``ASCII("omarchy-auth-preimage/v1") || 0x00 || JCS(A)`` where ``A`` contains the envelope format, signature format, key ID, signer role, algorithm, domain, context, payload type, payload version, schema-set digest, payload digest, the ``anti_transplant`` object (document ID, schema, payload type, payload version, schema-set digest, domain, context), and the complete payload. The stable ``document_id`` is never a content digest; the envelope-level ``payload_digest`` is the separately computed identity U-Boot binds into its durable record.

U-Boot's consumer follows the only construction path: ``strict_parse -> canonicalize -> verify(Canonical<T>, Trusted<TrustContext>, VerifiedClock, ExpectedContext) -> Trusted<T>``. A failed step yields no partial value and no authority. ``schema_set_digest`` is compared byte-for-byte with the schema-set digest compiled into the U-Boot boot binding; any other value is ``BINDING_INTEGRITY_FAILURE`` and the boot consumer treats the object as absent.

Authority resolution uses only the closed ``AuthorityRoleBinding`` records inside the F-03-supplied ``Trusted<TrustContext>``: ``binding_schema``, ``authority_id``, ``role``, ``actor_id``, ``account_id``, ``key_ids``, ``allowed_methods``, ``service_policy_id``, ``service_policy_digest``, ``issued_at``, ``expires_at``, and ``binding_digest``. The nine roles are ``board-admission``, ``manifest-release``, ``installer-planner``, ``owner-authorization``, ``ci-conformance``, ``qualification-lab``, ``boot-runtime``, ``dtb-authority``, and ``evidence-reader``. U-Boot resolves exactly two roles at boot: ``manifest-release`` for the slot manifest and ``boot-runtime`` for the boot-health core and the boot-success mark. A key ID or role string in an envelope is an authenticated claim to be checked against the binding, never a hint that grants anything.

``ExpectedContext`` is constructed by U-Boot for each verification with the exact row from the eight-row signing contract: ``payload_type``, ``payload_version``, ``domain``, ``context``, ``project_id``, ``repository_id``, ``slice_id``, ``operation``, ``board_id``, ``manifest_id``, ``manifest_digest``, ``schema_set_digest``, ``target_account_id``, ``target_account_binding``, ``target_identity_digests``, and ``policy_digest``. The fixed ``project_id``, ``repository_id``, and ``slice_id`` values for the boot rows are BLK-14. A mismatch on any non-null member is ``SIGNATURE_CONTEXT_MISMATCH``.

Signer role, scope, expiry, and replay are enforced in this order: envelope field equality, signing-row equality, authority binding resolution, key expiry and revocation epoch, payload ``expires_at``, then replay reservation against the durable record. Anti-transplant is enforced twice: once inside the signed preimage and once by U-Boot comparing the ``document_id``, ``manifest_digest``, board, slot, generation, lineage, and counter values in the payload against its own durable record.

Payload rows consumed at the U-Boot boundary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PROVISIONAL, pending coordinator approval. The domain, context, and role columns are quoted from the candidate's exhaustive eight-row signing contract.

.. list-table::
   :header-rows: 1
   :widths: 20 20 20 12 28

   * - Payload type
     - Domain
     - Context
     - Role
     - U-Boot consumer
   * - ``board-registry/v1``
     - ``omarchy-board-registry``
     - ``board-registry-publication``
     - ``board-admission``
     - Board identity source at boot; delivery form is BLK-11
   * - ``platform-manifest/v1``
     - ``omarchy-platform-manifest``
     - ``manifest-publication``
     - ``manifest-release``
     - Per-slot manifest; sole release-tuple authority
   * - ``installer-plan/v1``
     - ``omarchy-installer-plan``
     - ``installer-plan-proposal``
     - ``installer-planner``
     - Not consumed at boot; provisions control partitions (T-01)
   * - ``qualification-record/v1``
     - ``omarchy-qualification-record``
     - ``qualification-result``
     - ``qualification-lab``
     - Not consumed at boot; referenced by manifest ``qualification_bindings``
   * - ``boot-health/v1``
     - ``omarchy-boot-runtime``
     - ``boot-health-core``
     - ``boot-runtime``
     - Consumed on the boot after the attempt, from the success-mark container
   * - ``owner-approval/v1``
     - ``omarchy-owner-authorization``
     - ``installer-plan-execution``
     - ``owner-authorization``
     - Not consumed at boot; authorizes the provisioning mutation (T-01)
   * - ``boot-success-mark/v1``
     - ``omarchy-boot-runtime``
     - ``boot-success-marker``
     - ``boot-runtime``
     - Consumed on the boot after the attempt; sole success statement
   * - ``dtb-mutation-envelope/v1``
     - ``omarchy-dtb-authority``
     - ``dtb-mutation-authorization``
     - ``dtb-authority``
     - Not consumed by U-Boot; U-Boot performs no authorized DTB mutation

Exact paths bound by this design
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PROVISIONAL, pending coordinator approval. Paths are JSON paths into the verified envelope. U-Boot reads only the listed paths; every other path is verified for closure and then ignored.

.. list-table::
   :header-rows: 1
   :widths: 22 46 32

   * - Document
     - Path
     - Use at the U-Boot boundary
   * - ``platform-manifest/v1``
     - ``$.payload.document_id`` and envelope ``payload_digest``
     - Manifest identity bound into BCR-S09 and BCR-S08
   * - ``platform-manifest/v1``
     - ``$.payload.channel``
     - Must equal the compiled release channel (``stable`` for the release profile)
   * - ``platform-manifest/v1``
     - ``$.payload.board_targets[]``
     - Must contain the exact board ID; SoC-only targeting is rejected
   * - ``platform-manifest/v1``
     - ``$.payload.board_registry_digest``
     - Must equal the digest of the trusted board registry U-Boot used (BLK-11)
   * - ``platform-manifest/v1``
     - ``$.payload.components.boot_stack.artifacts[]``
     - Exact ``artifact_id``, ``kind``, ``media_type``, ``size_bytes``, ``content_digest`` for every file U-Boot loads or GRUB is permitted to load
   * - ``platform-manifest/v1``
     - ``$.payload.components.boot_stack.boot_check_profile``
     - ``profile_id``, ``profile_digest``, ``required_check_ids``, ``retry_limit``, ``failure_limit``, ``rollback_manifest_ids``; source of BCR-S04
   * - ``platform-manifest/v1``
     - ``$.payload.components.boot_stack.rollback``
     - ``previous_manifest_ids`` must name the current last-known-good manifest before a staged slot becomes PENDING
   * - ``platform-manifest/v1``
     - ``$.payload.rollback``
     - ``failure_attempt_limit`` and ``manifest_ids`` used to recompute ``rollback_set_digest`` (BCR-S16)
   * - ``platform-manifest/v1``
     - ``$.payload.components.linux_kernel.artifacts[]`` and ``$.payload.components.dtb_set.artifacts[]``
     - Kernel image, initramfs, and DTB digests that GRUB is required to re-check and U-Boot verifies before launch
   * - ``platform-manifest/v1``
     - ``$.payload.consumer_schema_set`` and ``$.payload.minimum_consumer_api``
     - Negotiated against the compiled boot binding; a lower consumer API is ``CROSS_DOCUMENT_MISMATCH``
   * - ``board-registry/v1``
     - ``$.payload.boards[i].board_id`` and ``$.payload.boards[i].identity_match.linux``
     - Root ``compatible`` list and ``model`` from the working device tree must match exactly one board
   * - ``board-registry/v1``
     - ``$.payload.boards[i].firmware.firmware_schema_id`` and ``firmware_schema_version``
     - Cross-checked against the manifest ``firmware_schema`` projection
   * - ``boot-health/v1``
     - ``$.payload.board_id``, ``manifest_id``, ``manifest_digest``, ``profile_id``, ``profile_digest``, ``lineage_id``, ``source_generation``
     - Must equal the pending slot descriptor and the trusted manifest
   * - ``boot-health/v1``
     - ``$.payload.slot.slot_id``, ``slot.slot_generation``, ``slot.boot_artifact_digest``
     - Must equal BCR-S07 and BCR-S10 of the pending slot
   * - ``boot-health/v1``
     - ``$.payload.attempt.counter``
     - Must equal BCR-S06 of the pending slot; any other value is ``BOOT_COUNTER_FAILURE``
   * - ``boot-health/v1``
     - ``$.payload.checks[]`` and ``$.payload.checks_digest``
     - Recomputed; every required check ID from the profile must be present exactly once with ``status = pass`` for success
   * - ``boot-health/v1``
     - ``$.payload.success`` and ``$.payload.fallback``
     - ``success`` is never trusted without a matching mark; ``fallback.decision = recover`` triggers T-07
   * - ``boot-success-mark/v1``
     - ``$.payload.core_digest``
     - Must equal ``D_core = sha256(ASCII("omarchy-boot-health-core/v1") || 0x00 || JCS(core payload))``
   * - ``boot-success-mark/v1``
     - ``$.payload.board_id`` through ``$.payload.attempt_counter``
     - Must equal the core and the pending slot descriptor field by field
   * - ``boot-success-mark/v1``
     - ``$.payload.marker_generation`` and ``$.payload.marker_replay_id``
     - Strictly greater than BCR-S12 and different from BCR-S13; recorded on accept
   * - ``boot-success-mark/v1``
     - ``$.payload.checks_digest`` and ``$.payload.rollback_set_digest``
     - Must equal the core value and the U-Boot recomputation from the trusted manifest
   * - ``boot-success-mark/v1``
     - ``$.payload.marked_at`` and ``$.payload.expires_at``
     - Freshness under BLK-06; the counter binding is the replay authority in every case
   * - ``boot-success-mark/v1``
     - ``$.payload.diagnostic_note``
     - Bounded to 2,048 UTF-8 bytes; never parsed for authority and never printed by U-Boot
   * - ``owner-approval/v1``
     - ``$.payload.scope`` target identities
     - The installer's provisioning of the two control partitions and the mark partition (T-01) must be inside the approved scope; U-Boot has no boot-time consumer
   * - ``qualification-record/v1``
     - ``$.payload.manifest.manifest_digest`` and ``$.payload.outcome``
     - Consumed by F-05 and F-07 before a stable manifest exists; U-Boot relies on ``channel`` and the manifest signature, not on re-reading qualification records

Limits the boot binding must enforce
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PROVISIONAL, pending coordinator approval. The complete canonical success-mark envelope is at most 4,096 bytes inclusive; the core and mark payloads are each at most 3,072 bytes; boot objects have depth at most 8, at most 32 properties, arrays at most 32 entries, and at most 32 checks. The complete core envelope maximum is not stated by the candidate and is BLK-13; the U-Boot container reserves 4,096 bytes for it and rejects a ratified bound above that as a design conflict requiring a new ruling rather than silently widening. ``Digest`` is ``sha256:`` plus 64 lowercase hexadecimal characters. ``SlotId`` is exactly ``slot-a``, ``slot-b``, or ``recovery``. ``BoardId`` is ``apple:`` plus a lowercase token of 1 to 64 bytes. ``Generation`` and counters are unsigned 64-bit values that never wrap or reset; the value 18,446,744,073,709,551,615 is a hard stop that returns ``BOOT_COUNTER_FAILURE`` before any increment.

Failure vocabulary used by the boot consumer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PROVISIONAL, pending coordinator approval. The boot HOLD set is exactly ``BOOT_MARKER_AUTH_FAILURE``, ``BOOT_CONTEXT_MISMATCH``, ``BOOT_COUNTER_FAILURE``, ``BOOT_REQUIRED_CHECK_FAILURE``, ``BOOT_FALLBACK_FAILURE``, and ``TRUST_BOUNDARY_FAILURE``. A HOLD code never produces success and never selects a slot that is not in the recomputed rollback set. Every other code encountered by U-Boot (``PARSE_SCHEMA_FAILURE``, ``UNKNOWN_FIELD``, ``DUPLICATE_SEMANTIC_KEY``, ``CANONICALIZATION_FAILURE``, ``SIGNATURE_CONTEXT_MISMATCH``, ``TRUST_FAILURE``, ``EXPIRY_OR_REPLAY_FAILURE``, ``MANIFEST_EXPIRY_FAILURE``, ``CROSS_DOCUMENT_MISMATCH``, ``DOCUMENT_ID_REUSE``, ``BINDING_INTEGRITY_FAILURE``, ``RESOURCE_LIMIT``) is a reject: the object is treated as absent. U-Boot maps these to the 16-bit local codes in `Local failure codes`_ for storage in the durable record; the mapping is one-to-one and the string code is what U-Boot prints.

Authority separation
--------------------

Four authorities are kept apart. No component holds more than one.

.. list-table::
   :header-rows: 1
   :widths: 24 24 52

   * - Authority
     - Sole holder
     - What the holder may and may not do
   * - Release-channel authority
     - F-07 promotion terminal
     - Alone copies an already-built candidate digest into the ``stable`` namespace. U-Boot never contacts a channel and never decides what is stable; it checks only that the immutable manifest staged in a slot carries ``channel`` equal to its compiled profile and verifies under ``manifest-release``.
   * - Boot selection authority
     - U-Boot ``omarchy release``
     - Alone chooses which verified slot is launched. Consumes an immutable candidate manifest; never edits it, never chooses a component version, never scans for alternatives.
   * - Boot-control transition authority
     - U-Boot ``omarchy release``
     - Alone writes any state transition of the two boot-control copies (T-02 through T-09, T-11, T-12). The OS never writes PENDING, ACCEPTED, or FAILED. The only non-U-Boot write is the pre-boot provisioning of an unkeyed record that cannot express acceptance (T-01).
   * - Boot success authority
     - ``boot-runtime`` signer inside the booted OS, evaluated by U-Boot
     - Emits a signed ``boot-health/v1`` core and a separately signed ``boot-success-mark/v1`` into the success-mark container. It cannot mutate boot-control copies through that container, and the container has no field that means accepted. U-Boot validates ``Trusted<BootSuccessMark>`` on the next boot and alone performs the PENDING to ACCEPTED transition.

Missing, stale, replayed, forged, mismatched, or oversize marks fail closed: the pending slot keeps consuming its bounded attempt budget, the last-known-good slot is never consumed by a mark failure, and no mark failure can change the last-known-good descriptor.

Durable boot-control record
---------------------------

Storage identity
~~~~~~~~~~~~~~~~

Two Omarchy-owned GPT partitions on the same NVMe disk as the Apple ESP hold one copy each of the boot-control record (BCR). Both partitions carry the partition type GUID BLK-01, the partition names ``OMARCHY-BCR-0`` and ``OMARCHY-BCR-1``, and a size of exactly 1,048,576 bytes. Only bytes 0 through 4,095 of each partition are ever read or written; the remainder is zero at provisioning and is never read. The two partitions are provisioned by the installer under an approved plan whose scope names their stable IDs (``partition:v1:sha256:...`` under the F-02 typed storage identity) and are never created, moved, resized, or deleted by U-Boot.

U-Boot resolves each copy by all of the following, in this order, and refuses the copy if any check fails: the NVMe device parent is ``apple,nvme-ans2``; ``gpt_verify_headers()`` succeeds for the disk; the disk GUID equals BCR-09 in the record being validated; the partition type GUID equals BLK-01; the unique partition GUID equals BCR-10 or BCR-11 for the copy index claimed by the record; the partition name equals the expected name for that index; the partition size equals 1,048,576 bytes; and exactly one partition satisfies the tuple. Resolution is repeated immediately before every write. A raw device path, environment value, glob, or first-match fallback is never accepted as the write target.

Record encoding
~~~~~~~~~~~~~~~

The record is exactly 4,096 bytes and is written as one aligned 4,096-byte write at logical block 0 of its partition. All multi-byte integers are little-endian. GPT GUID fields (BCR-09, BCR-10, BCR-11, BCR-12) hold the 16 bytes exactly as they appear in the GPT on-disk entry so that comparison to the partition table is bytewise. UUID fields originating from F-02 documents (BCR-08, BCR-S07 lineage, BCR-S13) hold the 16 bytes of the canonical lowercase textual form in RFC 4122 network order. Digest fields hold the 32 raw bytes decoded from the 64 hexadecimal characters. ASCII fields are NUL-terminated within the field and every byte after the terminator is zero. Every reserved byte is zero, and a non-zero reserved byte invalidates the copy. Each field has a stable identifier so that reviewers can count them mechanically.

Header region, bytes 0 to 255:

.. list-table::
   :header-rows: 1
   :widths: 9 8 6 9 22 46

   * - ID
     - Offset
     - Size
     - Type
     - Field
     - Bound
   * - BCR-01
     - 0
     - 16
     - u8[16]
     - ``magic``
     - Exactly the ASCII bytes ``OMARCHY-BCR-v1`` followed by two 0x00 bytes
   * - BCR-02
     - 16
     - 4
     - u32
     - ``record_length``
     - Exactly 4096
   * - BCR-03
     - 20
     - 2
     - u16
     - ``format_major``
     - Exactly 1; any other value is an unknown format and invalidates the copy
   * - BCR-04
     - 22
     - 2
     - u16
     - ``format_minor``
     - Exactly 0 for this format; a higher minor under major 1 invalidates the copy because no compatibility rule exists yet
   * - BCR-05
     - 24
     - 4
     - u32
     - ``copy_index``
     - 0 or 1; must equal the index of the partition the copy was read from
   * - BCR-06
     - 28
     - 4
     - u32
     - ``auth_algorithm``
     - 1 = ``sha256-binding/v1`` (unkeyed, non-release and provisioning only), 2 = ``hmac-sha256/v1`` (release); other values invalidate the copy
   * - BCR-07
     - 32
     - 8
     - u64
     - ``sequence``
     - 1 through 18,446,744,073,709,551,614; 0 and the maximum invalidate the copy; strictly increases by exactly 1 per commit
   * - BCR-08
     - 40
     - 16
     - uuid
     - ``control_set_id``
     - Installer-plan target identity for this control set (BLK-15); must be equal in both copies and in the mark container
   * - BCR-09
     - 56
     - 16
     - guid
     - ``disk_guid``
     - GPT disk GUID of the disk holding both copies
   * - BCR-10
     - 72
     - 16
     - guid
     - ``copy0_partition_guid``
     - Unique partition GUID of ``OMARCHY-BCR-0``
   * - BCR-11
     - 88
     - 16
     - guid
     - ``copy1_partition_guid``
     - Unique partition GUID of ``OMARCHY-BCR-1``
   * - BCR-12
     - 104
     - 16
     - guid
     - ``mark_partition_guid``
     - Unique partition GUID of the success-mark partition
   * - BCR-13
     - 120
     - 32
     - digest
     - ``schema_set_digest``
     - Must equal the schema-set digest compiled into the boot binding
   * - BCR-14
     - 152
     - 72
     - ascii
     - ``board_id``
     - ``apple:`` plus 1 to 64 token bytes, NUL padded; must equal the board resolved from the working device tree
   * - BCR-15
     - 224
     - 8
     - u64
     - ``source_generation``
     - Topology source generation from the installer's committed snapshot; 0 only in a provisioned-unaccepted record
   * - BCR-16
     - 232
     - 1
     - u8
     - ``selected_slot``
     - 1 = ``slot-a``, 2 = ``slot-b``, 3 = ``recovery``; 0 invalidates the copy
   * - BCR-17
     - 233
     - 1
     - u8
     - ``last_known_good_slot``
     - 0 = none, 1 = ``slot-a``, 2 = ``slot-b``; 3 is invalid because recovery is never last-known-good
   * - BCR-18
     - 234
     - 2
     - u16
     - ``record_failure_code``
     - One code from `Local failure codes`_; 0 = none
   * - BCR-19
     - 236
     - 4
     - u32
     - ``reserved_a``
     - 0
   * - BCR-20
     - 240
     - 8
     - i64
     - ``transition_time_unix``
     - Evidence only; seconds since the Unix epoch or 0 when unknown; never used for ordering or freshness
   * - BCR-21
     - 248
     - 8
     - u64
     - ``reserved_b``
     - 0

Slot descriptors: ``slot-a`` at offset 256, ``slot-b`` at offset 512, and ``recovery`` at offset 768. Each descriptor is 256 bytes with the following layout relative to its base. The 17 descriptor fields occur three times, once per descriptor.

.. list-table::
   :header-rows: 1
   :widths: 9 8 6 9 26 42

   * - ID
     - Offset
     - Size
     - Type
     - Field
     - Bound
   * - BCR-S01
     - +0
     - 1
     - u8
     - ``slot_state``
     - 0 = EMPTY, 1 = PENDING, 2 = ACCEPTED, 3 = FAILED, 4 = PINNED; PINNED only in the recovery descriptor; PENDING, ACCEPTED, and FAILED never in the recovery descriptor
   * - BCR-S02
     - +1
     - 1
     - u8
     - ``reserved_s0``
     - 0
   * - BCR-S03
     - +2
     - 2
     - u16
     - ``attempts_used``
     - 0 through ``attempt_limit``; 0 unless PENDING or FAILED
   * - BCR-S04
     - +4
     - 2
     - u16
     - ``attempt_limit``
     - 1 through 8, copied from ``boot_check_profile.retry_limit``; a profile value of 0 or above 8 is ``BOOT_FALLBACK_FAILURE`` and the slot cannot become PENDING
   * - BCR-S05
     - +6
     - 2
     - u16
     - ``slot_failure_code``
     - One code from `Local failure codes`_; 0 unless FAILED
   * - BCR-S06
     - +8
     - 8
     - u64
     - ``attempt_counter``
     - 0 = unstarted; strictly increasing within a lineage; equals the last counter durably reserved before a launch
   * - BCR-S07
     - +16
     - 8
     - u64
     - ``slot_generation``
     - 0 only when EMPTY; increases by exactly 1 each time the descriptor enters PENDING
   * - BCR-S08
     - +24
     - 32
     - digest
     - ``manifest_digest``
     - Envelope ``payload_digest`` of the slot manifest; zero only when EMPTY
   * - BCR-S09
     - +56
     - 32
     - digest
     - ``manifest_id_digest``
     - ``sha256`` of the ASCII bytes of ``$.payload.document_id``; fixed width regardless of the ``DocumentId`` grammar (BLK-07)
   * - BCR-S10
     - +88
     - 32
     - digest
     - ``boot_artifact_digest``
     - ``content_digest`` of the boot-stack artifact U-Boot executes for this slot (BLK-12)
   * - BCR-S11
     - +120
     - 16
     - uuid
     - ``lineage_id``
     - Allocated once per board, manifest, and slot generation (BLK-08); zero only when EMPTY
   * - BCR-S12
     - +136
     - 8
     - u64
     - ``accepted_marker_generation``
     - ``marker_generation`` of the mark consumed at acceptance; 0 unless ACCEPTED
   * - BCR-S13
     - +144
     - 16
     - uuid
     - ``accepted_marker_replay_id``
     - ``marker_replay_id`` of the mark consumed at acceptance; zero unless ACCEPTED
   * - BCR-S14
     - +160
     - 32
     - digest
     - ``accepted_mark_digest``
     - ``D_mark = sha256(ASCII("omarchy-boot-success-mark/v1") || 0x00 || JCS(mark payload))`` of the consumed mark; zero unless ACCEPTED
   * - BCR-S15
     - +192
     - 8
     - i64
     - ``pending_time_unix``
     - Evidence only; 0 when unknown
   * - BCR-S16
     - +200
     - 32
     - digest
     - ``rollback_set_digest``
     - Recomputed from the trusted manifest at the PENDING commit; zero only when EMPTY or PINNED
   * - BCR-S17
     - +232
     - 24
     - u8[24]
     - ``reserved_s1``
     - 0

Bytes 1,024 through 4,047 are reserved and zero. Trailer region, bytes 4,048 to 4,095:

.. list-table::
   :header-rows: 1
   :widths: 9 8 6 9 22 46

   * - ID
     - Offset
     - Size
     - Type
     - Field
     - Bound
   * - BCR-T01
     - 4048
     - 8
     - u64
     - ``trailer_sequence``
     - Must equal BCR-07; a mismatch is a torn write and invalidates the copy
   * - BCR-T02
     - 4056
     - 32
     - u8[32]
     - ``auth_tag``
     - Algorithm 1: ``sha256(ASCII("omarchy-bcr-binding/v1") || 0x00 || bytes[0..4055])``; algorithm 2: ``HMAC-SHA256(OMARCHY_BCR_AUTH_KEY, same preimage)`` using ``lib/sha256.c`` ``sha256_hmac()``
   * - BCR-T03
     - 4088
     - 4
     - u32
     - ``reserved_t``
     - 0
   * - BCR-T04
     - 4092
     - 4
     - u32
     - ``crc32c``
     - CRC-32C (``lib/crc32c.c``) over bytes 0 through 4,091

CRC versus authenticated fields: the CRC exists to detect torn writes and media errors and grants nothing. The authenticated region is bytes 0 through 4,055, which includes every semantic field and the trailer sequence. Under algorithm 2 the tag is a MAC keyed by BLK-03 and is the record's authentication. Under algorithm 1 the tag is an unkeyed binding digest that detects accidental corruption and copy transplant only. The release profile requires algorithm 2 and a valid MAC for any record that expresses ACCEPTED, FAILED, a non-zero ``last_known_good_slot``, or ``attempts_used`` above 0; an unkeyed record may express only the provisioned-unaccepted shape defined under T-01. This is the structural rule that makes the OS incapable of writing acceptance: the OS never holds the key, and the only unkeyed shape U-Boot accepts cannot say accepted.

The two copies of one commit are identical except in ``copy_index``, ``auth_tag``, and ``crc32c``. Equality comparison between copies therefore excludes exactly those three fields.

Local failure codes
~~~~~~~~~~~~~~~~~~~

The 16-bit codes stored in BCR-18 and BCR-S05 are closed. Each maps one-to-one to the F-02 string it reports, and U-Boot prints the string.

.. list-table::
   :header-rows: 1
   :widths: 8 30 62

   * - Code
     - Reported string
     - Meaning at the U-Boot boundary
   * - 0
     - none
     - No failure recorded
   * - 1
     - ``BOOT_FALLBACK_FAILURE``
     - Attempt budget exhausted without an accepted mark, or a fallback target outside the rollback set
   * - 2
     - ``BOOT_MARKER_AUTH_FAILURE``
     - Mark or core signature, role, domain, or context failed
   * - 3
     - ``BOOT_CONTEXT_MISMATCH``
     - Mark, core, or manifest tuple differs from the pending descriptor
   * - 4
     - ``BOOT_COUNTER_FAILURE``
     - Counter or generation reset, reuse, wrap, mismatch, or maximum reached
   * - 5
     - ``BOOT_REQUIRED_CHECK_FAILURE``
     - Required check absent, failed, duplicated, or ``checks_digest`` mismatch
   * - 6
     - ``TRUST_BOUNDARY_FAILURE``
     - Untrusted local record or partial read at a trust seam
   * - 7
     - ``CROSS_DOCUMENT_MISMATCH``
     - Manifest, registry, or artifact identity disagreement
   * - 8
     - ``BINDING_INTEGRITY_FAILURE``
     - Schema-set digest or consumer API disagreement
   * - 9
     - ``MANIFEST_EXPIRY_FAILURE``
     - Slot manifest expired under BLK-06
   * - 10
     - ``EXPIRY_OR_REPLAY_FAILURE``
     - Marker generation or replay ID already consumed
   * - 11
     - ``SIGNATURE_CONTEXT_MISMATCH``
     - Envelope row disagreement for the slot manifest
   * - 12
     - ``RESOURCE_LIMIT``
     - Envelope, container, or property beyond a fixed bound
   * - 13
     - ``PARSE_SCHEMA_FAILURE``
     - Strict parse, canonicalization, unknown field, or duplicate key on any consumed object
   * - 14
     - ``OMARCHY_ARTIFACT_DIGEST_MISMATCH``
     - A manifest-declared artifact in the slot has a different size or ``content_digest``
   * - 15
     - ``OMARCHY_ARTIFACT_MISSING``
     - A manifest-declared artifact required for the boot path is absent from the slot
   * - 16
     - ``OMARCHY_BCR_DIVERGENT``
     - Both copies valid with equal sequence and different content
   * - 17
     - ``OMARCHY_BCR_COMMIT_FAILED``
     - A two-copy commit could not be completed and read back
   * - 18
     - ``OMARCHY_BOARD_MISMATCH``
     - Working device tree does not resolve to exactly one registry board or does not equal BCR-14
   * - 19
     - ``OMARCHY_LKG_INVALID``
     - Last-known-good slot content no longer equals its accepted descriptor

The codes 14 through 19 are U-Boot-local diagnostic codes for conditions the F-02 vocabulary does not name at this boundary. They are never written into an F-02 payload; the OS-side writer maps a U-Boot local code it observes in the BootContext to ``TRUST_BOUNDARY_FAILURE`` if it must report one.

Slot states and transitions
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The five slot states are EMPTY, PENDING, ACCEPTED, FAILED, and PINNED. The record-level fields ``selected_slot`` and ``last_known_good_slot`` are derived by the transitions; they are never set independently. Every transition except T-01 is performed only by U-Boot inside ``omarchy release`` and is committed with the two-copy procedure before any launch.

.. list-table::
   :header-rows: 1
   :widths: 7 16 23 54

   * - ID
     - Actor
     - From and to
     - Precondition, effect, and what is committed
   * - T-01
     - Installer (Linux stage under an approved plan)
     - none to provisioned-unaccepted
     - Writes both copies with ``sequence = 1``, ``auth_algorithm = 1``, ``slot-a`` PENDING with ``attempts_used = 0``, ``attempt_counter = 0``, ``slot_generation = 1``, the staged manifest digests, ``slot-b`` EMPTY, ``recovery`` PINNED with the recovery manifest digest, ``last_known_good_slot = 0``, ``selected_slot = 1``. Any other content in an unkeyed record is invalid. Also permitted from a recovery OS only after both keyed copies have been destroyed.
   * - T-02
     - U-Boot
     - unkeyed to keyed
     - Folded into the first U-Boot commit: the record is rewritten with ``auth_algorithm = 2`` and a fresh MAC. In a non-release profile the record stays at algorithm 1.
   * - T-03
     - U-Boot
     - EMPTY, FAILED, or non-last-known-good ACCEPTED to PENDING (stage detection)
     - The slot's on-disk manifest verifies as ``Trusted<PlatformManifest>``, targets the board, carries the required channel, names the current last-known-good manifest ID in ``components.boot_stack.rollback.previous_manifest_ids`` (or ``last_known_good_slot = 0``), every boot-path artifact verifies, ``retry_limit`` is 1 through 8, and ``payload_digest`` differs from a FAILED descriptor's ``manifest_digest``. Effect: ``slot_generation + 1``, new ``lineage_id``, ``attempt_counter = 0``, ``attempts_used = 0``, ``attempt_limit``, digests, ``rollback_set_digest``, ``selected_slot`` set to this slot. The previous descriptor content is tombstoned by overwrite (T-12).
   * - T-04
     - U-Boot
     - PENDING to PENDING (attempt consumption)
     - ``selected_slot`` is PENDING, ``attempts_used < attempt_limit``, ``attempt_counter`` below the maximum. Effect: ``attempts_used + 1``, ``attempt_counter + 1``, ``pending_time_unix``. Committed before the BootContext is installed and before any launch.
   * - T-05
     - U-Boot
     - PENDING to ACCEPTED
     - A valid core and mark tuple for exactly ``attempt_counter`` passed every check in `Success-mark transport`_. Effect: state ACCEPTED, ``last_known_good_slot`` set to this slot, BCR-S12, BCR-S13, BCR-S14 recorded, ``attempts_used`` retained as evidence. The former last-known-good slot remains ACCEPTED but is no longer last-known-good and becomes eligible for T-03. Committed before the mark container is cleared.
   * - T-06
     - U-Boot
     - PENDING to FAILED (budget exhausted)
     - ``attempts_used = attempt_limit`` and no valid mark. Effect: state FAILED, ``slot_failure_code = 1``, ``selected_slot`` set to ``last_known_good_slot`` when non-zero, otherwise 3. The FAILED descriptor keeps its ``manifest_digest`` so the same candidate cannot be retried (rollback recursion guard).
   * - T-07
     - U-Boot
     - PENDING to FAILED (explicit OS failure)
     - A ``Trusted<BootHealthCore>`` for exactly ``attempt_counter`` carries ``success = false`` and ``fallback.decision = recover`` with ``target_slot`` in the recomputed rollback set. Effect as T-06 with ``slot_failure_code`` from the core ``fallback.failure_code`` mapping. A core with ``decision = hold`` leaves the slot PENDING and the next boot performs T-04.
   * - T-08
     - U-Boot
     - PENDING to FAILED (pre-launch validation failure)
     - The pending slot's manifest is missing, unsigned, expired, not the recorded ``manifest_digest``, or any boot-path artifact fails size or digest. Effect as T-06 with codes 7, 9, 11, 13, 14, or 15.
   * - T-09
     - U-Boot
     - record-level only
     - The last-known-good slot's on-disk manifest ``payload_digest`` no longer equals its ACCEPTED descriptor or a boot-path artifact fails. Effect: ``record_failure_code = 19``, ``selected_slot = 3``; the descriptor is not erased. If the commit fails, the recovery launch proceeds without a write.
   * - T-10
     - U-Boot
     - none
     - Manual diagnostic boot (``omarchy try``) and recovery boot (``omarchy recovery`` or automatic recovery selection). No write of either copy in any case.
   * - T-11
     - U-Boot
     - none (read-repair)
     - A stale or invalid copy is rewritten from the selected valid record with its own ``copy_index``, tag, and CRC. ``sequence`` is unchanged. Performed after selection and before any transition commit; a repair failure is logged and does not stop a boot that requires no transition.
   * - T-12
     - U-Boot
     - descriptor overwrite (tombstone)
     - Part of T-03: the prior lineage in that descriptor is permanently tombstoned because ``slot_generation`` increased; an older mark for the tombstoned lineage can never match again.

State consistency rules checked on every read, any violation invalidating the copy: ``selected_slot`` names a descriptor that is not EMPTY; a PENDING or FAILED descriptor has ``attempt_limit`` in range and ``attempts_used`` within it; ``last_known_good_slot`` names an ACCEPTED descriptor; at most one of ``slot-a`` and ``slot-b`` is PENDING; the recovery descriptor is EMPTY or PINNED; an ACCEPTED descriptor has non-zero BCR-S12, BCR-S13, and BCR-S14; an unkeyed record has exactly the T-01 shape.

Two-copy selection and recovery
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On every boot U-Boot reads both copies before doing anything else. A copy is valid only if every check in `Record encoding`_, `Local failure codes`_, and the state consistency rules passes, including the partition identity checks in `Storage identity`_. Selection precedence is: keyed valid over unkeyed valid; higher ``sequence`` over lower; and among equal sequences, bytewise equality outside the three copy-specific fields. Timestamps never participate in selection.

.. list-table::
   :header-rows: 1
   :widths: 8 34 58

   * - ID
     - Observation
     - Decision
   * - C2-01
     - Both valid, equal sequence, equal content
     - Use the record; no repair
   * - C2-02
     - Both valid, equal sequence, different content
     - Code 16; no write of either copy; recovery slot if it verifies, otherwise HALT
   * - C2-03
     - Both valid, sequences differ by exactly 1
     - Use the higher; T-11 repairs the lower (interrupted commit between copies)
   * - C2-04
     - Both valid, sequences differ by more than 1
     - Use the higher; T-11 repairs the lower; diagnostic note that one copy lagged more than one commit
   * - C2-05
     - One valid, the other fails any validity check
     - Use the valid copy; T-11 repairs the other
   * - C2-06
     - One valid, the other partition cannot be resolved by the identity tuple
     - Use the valid copy; no repair. Commits are impossible because a commit needs both copies; a boot that requires a transition is refused, the last-known-good slot boots without a write, otherwise recovery
   * - C2-07
     - Zero valid, both partitions resolved
     - No write; recovery slot if its manifest verifies against the board and the PINNED digest is unavailable, so verification is signature and board only; otherwise HALT
   * - C2-08
     - GPT invalid or neither partition resolved
     - No write; HALT with the GPT failure printed; the ESP cannot be trusted either
   * - C2-09
     - A copy whose ``copy_index`` or partition GUID does not match the partition it was read from
     - That copy is invalid (transplant); the case reduces to C2-05 or C2-07
   * - C2-10
     - A keyed valid copy and an unkeyed valid copy
     - The keyed copy wins regardless of sequence; the unkeyed copy is treated as invalid and repaired. Re-provisioning therefore requires destroying both keyed copies first, which is an explicit recovery action and never an accidental downgrade
   * - C2-11
     - Power loss during the copy 0 write of a commit
     - Copy 0 is torn and invalid, copy 1 holds the previous state; C2-05 selects the previous state; the transition was not committed and nothing was launched
   * - C2-12
     - Power loss after copy 0 and during the copy 1 write
     - Copy 0 holds the new state, copy 1 is torn; C2-05 selects the new state and repairs copy 1; the transition is committed because launch waits for both copies
   * - C2-13
     - Power loss during a T-11 repair
     - The repaired copy is torn; next boot is C2-05 again with the same outcome
   * - C2-14
     - Both copies valid and both unkeyed with the T-01 shape
     - Accepted only if no keyed copy exists; the first U-Boot commit performs T-02

Commit procedure, executed for every transition in T-02 through T-09:

1. Re-resolve both partitions with the full identity tuple; abort with code 17 and no write if either fails.
2. Build the complete new record in memory with ``sequence = selected sequence + 1``; compute the trailer sequence, tag, and CRC for copy 0.
3. Write 4,096 bytes at block 0 of copy 0; issue a device flush (NI-08); read the block back and compare bytewise; abort with code 17 on any mismatch, leaving copy 1 authoritative.
4. Recompute tag and CRC for copy 1; write, flush, read back, compare; on failure the commit is still durable in copy 0 and the next boot repairs copy 1, but the current boot treats the commit as complete only if the transition is T-05 or T-06 through T-09; for T-04 the launch is refused because an attempt must be durably counted in both copies before the payload runs.
5. Only after step 4 completes does U-Boot install the BootContext and launch.

Writer serialization: U-Boot runs single-threaded, and the writer keeps a static in-progress flag; a re-entrant call returns code 17 without writing. Only ``omarchy release`` reaches the writer. The diagnostic subcommands, the EFI runtime, and any other command have no path to it. Alternate-copy ordering is fixed (copy 0 then copy 1) so that every interrupted state is one of C2-11 through C2-13.

Power-loss points that the sandbox suite must inject, each with the expected next-boot case: before step 1 (previous state), during step 3 (C2-11), between steps 3 and 4 (C2-03), during step 4 (C2-12), after step 5 before the payload runs (the attempt is consumed, C2-01), during GRUB (attempt consumed, no mark), during Linux before the mark write (no mark), during the mark container write (SM-04), after the mark write before the next boot (SM-16 on the next boot), during the T-05 commit (C2-11 or C2-12 with the mark still present, SM-17 on the following boot), and during the mark clear (SM-17).

Success-mark transport
----------------------

The success mark and the boot-health core travel in a third Omarchy-owned GPT partition, the boot success-mark container (BSM). The OS writes it; U-Boot reads, validates, and clears it. It has no state field that means accepted, so writing it cannot accept a slot.

Container identity and encoding
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The partition carries type GUID BLK-02, the name ``OMARCHY-BSM``, and a size of exactly 1,048,576 bytes; its unique GUID is recorded in BCR-12 and is part of the BCR authenticated region. Only bytes 0 through 12,287 are used: a 4,096-byte header block, a 4,096-byte core region, and a 4,096-byte mark region. All integers are little-endian.

.. list-table::
   :header-rows: 1
   :widths: 9 8 6 9 26 42

   * - ID
     - Offset
     - Size
     - Type
     - Field
     - Bound
   * - BSM-01
     - 0
     - 16
     - u8[16]
     - ``magic``
     - Exactly ``OMARCHY-BSM-v1`` followed by two 0x00 bytes
   * - BSM-02
     - 16
     - 4
     - u32
     - ``record_length``
     - Exactly 12288
   * - BSM-03
     - 20
     - 2
     - u16
     - ``format_major``
     - Exactly 1
   * - BSM-04
     - 22
     - 2
     - u16
     - ``format_minor``
     - Exactly 0
   * - BSM-05
     - 24
     - 4
     - u32
     - ``container_state``
     - 1 = WRITTEN, 2 = CONSUMED; other values make the container absent
   * - BSM-06
     - 28
     - 4
     - u32
     - ``producer_id``
     - 1 = OS boot-health writer, 2 = U-Boot consumer; evidence only, never authority
   * - BSM-07
     - 32
     - 8
     - u64
     - ``write_sequence``
     - OS-incremented per write; evidence only
   * - BSM-08
     - 40
     - 4
     - u32
     - ``core_length``
     - 1 through 4096 when WRITTEN
   * - BSM-09
     - 44
     - 4
     - u32
     - ``mark_length``
     - 1 through 4096 when WRITTEN
   * - BSM-10
     - 48
     - 4
     - u32
     - ``core_crc32c``
     - CRC-32C over the first ``core_length`` bytes of the core region
   * - BSM-11
     - 52
     - 4
     - u32
     - ``mark_crc32c``
     - CRC-32C over the first ``mark_length`` bytes of the mark region
   * - BSM-12
     - 56
     - 8
     - i64
     - ``written_time_unix``
     - Evidence only
   * - BSM-13
     - 64
     - 8
     - u64
     - ``consumed_attempt_counter``
     - Written by U-Boot when it sets CONSUMED; 0 when WRITTEN
   * - BSM-14
     - 72
     - 16
     - uuid
     - ``control_set_id``
     - Must equal BCR-08
   * - BSM-15
     - 88
     - 4004
     - u8[4004]
     - ``reserved``
     - 0
   * - BSM-16
     - 4092
     - 4
     - u32
     - ``header_crc32c``
     - CRC-32C over bytes 0 through 4,091 of the header block
   * - BSM-17
     - 4096
     - 4096
     - bytes
     - ``core_region``
     - Canonical UTF-8 ``omarchy-signed/v1`` envelope of the ``boot-health/v1`` core, no trailing newline, zero padded
   * - BSM-18
     - 8192
     - 4096
     - bytes
     - ``mark_region``
     - Canonical UTF-8 ``omarchy-signed/v1`` envelope of the ``boot-success-mark/v1`` payload, at most 4,096 bytes inclusive, zero padded

Canonical bytes are the exact JCS envelope bytes whose signature was produced by the ``boot-runtime`` key; the container adds no framing inside the regions. Authentication is entirely the Ed25519 envelope signature verified against the ``boot-runtime`` role binding in the F-03 trust context that U-Boot carries (BLK-05); the container's CRCs are for torn-write detection only.

Producer procedure (OS side, owned by the Linux consumer under K-01 and the health writer under P-05): construct ``Trusted<BootContext>`` from the firmware handoff as described in `BootContext transport`_; run the manifest-declared required checks; sign the core; sign the mark with ``attempt_counter`` equal to the BootContext counter, ``marker_generation`` greater than the previous accepted generation for the lineage, and a fresh ``marker_replay_id``; write the core region with a single aligned 4,096-byte write and flush; write the mark region the same way and flush; write the header block with ``container_state = 1`` and both region CRCs and flush; read back all three blocks and compare. The header is written last so it never references bytes that were not durably written. The writer refuses to run when the BootContext is absent or invalid, so a diagnostic boot produces no container write.

Consumer procedure (U-Boot, inside ``omarchy release`` and only after two-copy selection): read the three blocks; validate the header; validate region CRCs; verify the mark envelope, then the core envelope, under the boot binding; evaluate against the pending descriptor; commit T-05, T-07, or nothing; then clear the container by writing the header block with ``container_state = 2``, ``consumed_attempt_counter``, and a fresh header CRC, followed by a flush. The regions are left intact as evidence for ``omarchy status`` until the OS next writes. The order BCR commit first, clear second makes a power loss between them idempotent (SM-17).

Replay prevention needs no nonce field in the mark because the F-02 mark already carries ``attempt_counter``, ``marker_generation``, and ``marker_replay_id``. The counter is unique per attempt, durably reserved by T-04 before the payload runs, and never reused across a lineage or after a tombstone. A mark for any counter other than the pending descriptor's exact value is dead on arrival.

Privacy bounds: the container carries no field other than the two envelopes; ``diagnostic_note`` is bounded to 2,048 bytes, is never printed by U-Boot, and the OS writer is required by the F-02 privacy rules to exclude serials, hostnames, usernames, paths, addresses, and secrets from it. ``omarchy status`` prints the mark's digest, counter, generation, and decision only.

Mark evaluation cases
~~~~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 8 42 50

   * - ID
     - Observation
     - Decision
   * - SM-01
     - Mark partition unresolved or unreadable
     - Mark absent; no acceptance; no clear
   * - SM-02
     - Header magic, length, version, or header CRC invalid
     - Absent; container rewritten as CONSUMED only if the partition identity resolved
   * - SM-03
     - ``container_state = 2``
     - Absent; nothing to do
   * - SM-04
     - WRITTEN with a region CRC mismatch (torn write)
     - Absent; clear
   * - SM-05
     - WRITTEN with a length of 0 or above 4,096
     - ``RESOURCE_LIMIT``; absent; clear
   * - SM-06
     - Mark envelope fails strict parse, canonicalization, schema closure, or schema-set digest
     - Reject code; absent; clear
   * - SM-07
     - Mark signature, key, role, domain, or context fails
     - ``BOOT_MARKER_AUTH_FAILURE``; hold; clear
   * - SM-08
     - Mark board, manifest ID, manifest digest, profile, slot, slot generation, lineage, or source generation differs from the pending descriptor or trusted manifest
     - ``BOOT_CONTEXT_MISMATCH``; hold; clear
   * - SM-09
     - Mark ``attempt_counter`` differs from BCR-S06 of the pending descriptor
     - ``BOOT_COUNTER_FAILURE``; hold; clear
   * - SM-10
     - ``marker_generation`` not greater than BCR-S12 or ``marker_replay_id`` equal to BCR-S13
     - ``EXPIRY_OR_REPLAY_FAILURE``; reject; clear
   * - SM-11
     - Mark valid but core absent, torn, or failing verification
     - No success (missing core); hold; clear
   * - SM-12
     - Core valid with ``success = false`` and ``fallback.decision = hold``
     - Slot stays PENDING; the next T-04 proceeds; clear
   * - SM-13
     - Core valid with ``success = false``, ``decision = recover``, target in the rollback set
     - T-07; clear
   * - SM-14
     - Mark ``core_digest`` differs from the recomputed ``D_core``
     - ``BOOT_CONTEXT_MISMATCH``; hold; clear
   * - SM-15
     - ``checks_digest`` or ``rollback_set_digest`` differs from U-Boot's recomputation, or a required check is absent, duplicated, or not ``pass``
     - ``BOOT_REQUIRED_CHECK_FAILURE`` or ``BOOT_FALLBACK_FAILURE``; hold; clear
   * - SM-16
     - Every check passes and the selected descriptor is PENDING at that counter
     - T-05 committed, then clear
   * - SM-17
     - Valid mark whose counter equals an ACCEPTED descriptor's last counter (power loss after T-05 before clear)
     - Idempotent; clear only
   * - SM-18
     - Valid mark but no PENDING descriptor at that counter (FAILED, tombstoned, or diagnostic boot)
     - Ignored; clear
   * - SM-19
     - ``control_set_id`` differs from BCR-08
     - Absent (transplanted container); clear
   * - SM-20
     - Core ``fallback.target_slot`` outside the recomputed rollback set or rollback set empty when recovery is required
     - ``BOOT_FALLBACK_FAILURE``; hold; clear
   * - SM-21
     - ``marked_at`` or ``expires_at`` outside the freshness policy
     - Governed by BLK-06; until ratified U-Boot records the values as evidence and relies on the counter binding; a ratified clock rule replaces this row without changing any other row

A hold decision never consumes the last-known-good slot: the pending slot either continues its bounded budget (T-04) or fails (T-06, T-07, T-08), and only then is the last-known-good slot selected.

Release boot path
-----------------

Architecture and launcher decision
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The required release architecture is the program's ``m1n1 stage 1 -> m1n1 stage 2 + DT -> U-Boot -> GRUB -> kernel`` chain. This design chooses GRUB as the only release kernel launcher. Direct kernel boot from U-Boot (``booti``, ``bootm``, ``bootefi`` of a kernel EFI stub, or any bootflow method) is diagnostic and non-release: it is compiled out of the release profile, and even in a diagnostic profile it never installs a release BootContext, so it can never produce an accepted slot.

U-Boot's entry inputs from the opaque predecessor are exactly: a device-tree blob at the address the predecessor supplies; the root ``compatible`` and ``model`` values; ``/chosen`` properties that the existing board code already consumes (``asahi,efi-system-partition`` and ``/chosen/framebuffer``); and a memory map that does not overlap U-Boot, the DT, or the framebuffer. The human-signed predecessor envelope (BLK-10) must attest those inputs. U-Boot validates them at its boundary and makes no claim about how they are produced.

Sequence of the release command
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Resolve the board: read root ``compatible`` and ``model`` from the working device tree, match exactly one ``board-registry/v1`` board under ``identity_match.linux`` (delivery form BLK-11), and derive ``board_id``. Zero or more than one match is code 18 and HALT.
2. Resolve storage: locate the Apple ESP by the ``asahi,efi-system-partition`` UUID only. The current first-ESP fallback in ``asahi_esp_devpart()`` is excluded from the release profile (NI-13). Resolve the two BCR partitions and the BSM partition by the full identity tuple.
3. Read and select the BCR per `Two-copy selection and recovery`_; perform T-11 if applicable.
4. Consume and evaluate the BSM per `Mark evaluation cases`_; commit T-05 or T-07 if applicable; clear the container.
5. Stage detection: for each of ``slot-a`` and ``slot-b`` that is not the pending or last-known-good slot, evaluate T-03. At most one slot may become PENDING per boot.
6. Select: if ``selected_slot`` is PENDING and ``attempts_used < attempt_limit``, commit T-04; if ``attempts_used = attempt_limit``, commit T-06 and re-select; if ``selected_slot`` is ACCEPTED, verify it (T-09 on failure); if ``selected_slot`` is 3, verify the recovery manifest against the PINNED digest.
7. Verify the slot: parse and verify the slot manifest as ``Trusted<PlatformManifest>``; compare ``payload_digest`` to BCR-S08; check channel, board target, registry digest, consumer API; read every boot-path artifact declared under ``components.boot_stack.artifacts``, ``components.linux_kernel.artifacts``, and ``components.dtb_set.artifacts`` that the GRUB configuration names, compare size and ``content_digest``; on failure commit T-08 and re-select from step 6 at most twice (pending, then last-known-good, then recovery), then HALT.
8. Install the BootContext into a working copy of the device tree per `BootContext transport`_ (release mode only; T-10 boots install the diagnostic marker instead).
9. Launch: call ``efi_binary_run()`` with the verified GRUB image bytes already in memory, the working device tree, and no initrd. GRUB then loads only the files named by the verified configuration.
10. If ``efi_binary_run()`` returns, the attempt is already consumed; print the return status, and reset. The next boot continues the bounded budget.

Steps 1 through 8 complete before any GRUB byte executes. No step reads an environment variable for authority; the command reads the control device tree through ``gd->fdt_blob`` and the block devices through the driver model.

GRUB artifact identity and configuration binding
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The slot layout on the ESP is fixed: ``EFI/OMARCHY/slots/A/``, ``EFI/OMARCHY/slots/B/``, and ``EFI/OMARCHY/recovery/``. Each directory contains ``manifest.json`` (the complete ``omarchy-signed/v1`` envelope of the slot manifest) and the artifact files named by that manifest. The file name of each artifact inside the slot is exactly its manifest ``artifact_id`` with the ``artifact:`` prefix removed; the manifest is the only mapping from role to file. The exact artifact IDs and media types for the GRUB image, the GRUB configuration, the U-Boot image, the kernel image, the initramfs, and the DTB set are BLK-09. Until ratified, the roles are referred to here by manifest ``kind`` and position, never by an invented file name.

U-Boot identifies the GRUB image as the single ``components.boot_stack.artifacts[]`` entry whose ``kind = boot-image/v1`` and whose ``media_type`` is the ratified GRUB EFI media type (BLK-09), and identifies the GRUB configuration as the single entry whose ``media_type`` is the ratified GRUB configuration media type (BLK-09). Both are loaded into memory and digest-verified before launch; the configuration is passed to GRUB by being present in the slot directory at the path GRUB is built to read, and GRUB is built with its prefix fixed to the slot directory so that no global ``grub.cfg`` outside the slot is consulted.

The release GRUB configuration is generated by the candidate assembler (F-05) from the manifest and is therefore itself manifest-bound. It is required to: name only files inside its own slot directory; check every file it loads with GRUB's built-in SHA-256 command against the manifest ``content_digest`` values embedded in the configuration; never invoke ``devicetree``, ``chainloader``, ``search``, ``configfile``, or ``source``; never expose an interactive menu, editor, or command line in release; and execute ``reboot`` on any check failure so that the consumed attempt terminates deterministically instead of hanging. GRUB is not a slot state machine, never reads ``BootNext`` or ``BootOrder``, and never chooses a cross-slot kernel.

BootContext transport
~~~~~~~~~~~~~~~~~~~~~

There is exactly one BootContext transport: three properties under ``/chosen`` in the device tree that U-Boot installs as the ``EFI_FDT_GUID`` configuration table via ``efi_install_fdt()``.

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Property
     - Value
   * - ``/chosen/omarchy,boot-mode``
     - NUL-terminated string ``release`` or ``diagnostic-untrusted``; always present when U-Boot launched the payload
   * - ``/chosen/omarchy,boot-context``
     - Canonical JCS UTF-8 bytes of the sealed ``boot-context/v1`` record, at most 1,024 bytes, no trailing NUL; present only in ``release`` mode
   * - ``/chosen/omarchy,boot-context-sha256``
     - 32 bytes: ``sha256(ASCII("omarchy-boot-context-dt/v1") || 0x00 || boot-context bytes)``; present only in ``release`` mode

PROVISIONAL, pending coordinator approval: the sealed record fields are exactly ``context_schema = boot-context/v1``, ``board_id``, ``manifest_id``, ``manifest_digest``, ``lineage_id``, ``slot_id``, ``slot_generation``, ``attempt_counter``, ``source_generation``, ``atomic_record_digest``, ``lineage_source_digest``, and ``provenance`` with ``source_kind = atomic-boot-journal/v1``, ``source_api_version``, and ``storage_generation``. U-Boot fills them from the committed BCR: ``atomic_record_digest = sha256(bytes 0..4055 of the selected copy)`` (the authenticated region, computable by the OS without the MAC key), ``lineage_source_digest = sha256(ASCII("omarchy-boot-lineage-record/v1") || 0x00 || bytes 0..4055)``, and ``storage_generation = sequence``. The BootContext is not an authenticated payload; its authentication is the firmware channel itself, and its authority is enforced by U-Boot's cross-check on the next boot.

Installation by U-Boot: after the T-04 commit, copy the control device tree, add the three properties, and pass the copy to ``efi_binary_run()``. ``efi_install_fdt()`` then copies it again and runs ``image_setup_libfdt()``; the ``bootargs`` fixup reads the environment, which in the release profile has no ``bootargs`` variable and no writable store, so the fixup adds nothing. The properties survive those fixups because ``ft_board_setup()`` and ``fdt_chosen()`` only add or replace their own named properties.

Preservation by GRUB: GRUB is required to forward the ``EFI_FDT_GUID`` table it received to the kernel's EFI stub, adding only its own ``/chosen`` properties for the initramfs and command line. This is the behavior expected of the GRUB arm64 EFI Linux loader when ``devicetree`` is not used, and it is exactly what experiment E-01 must prove for the pinned GRUB artifact before any implementation promotion. GRUB has no validation role and no ability to make a context trustworthy; it can only preserve or damage it.

Validation by the Linux consumer (K-01 owned): read the three properties from ``/proc/device-tree/chosen``; require ``boot-mode = release``; recompute and compare the SHA-256 property; strictly parse the record under the bounded boot binding; read both BCR copies read-only from the control partitions, select by the same precedence, and require ``sha256(bytes 0..4055)`` of the selected copy to equal ``atomic_record_digest``; require the descriptor tuple for ``slot_id`` to equal every record field; construct ``Trusted<AtomicBootRecord>`` and then ``Trusted<BootContext>`` through the only constructor. Any failure is ``TRUST_BOUNDARY_FAILURE``, the health writer refuses to run, and no container write occurs.

Rejection by component for each failure:

.. list-table::
   :header-rows: 1
   :widths: 30 22 48

   * - Failure
     - Rejecting component
     - Deterministic outcome
   * - ``boot-mode`` absent or not ``release``
     - Linux consumer
     - No mark; pending budget continues in U-Boot
   * - ``boot-context`` absent (stripped by GRUB or a foreign loader)
     - Linux consumer
     - No mark; budget continues; E-01 failure if it happens with the pinned GRUB
   * - ``boot-context`` bytes mutated or truncated
     - Linux consumer via the SHA-256 property and strict parse
     - No mark
   * - Both properties consistently forged in Linux
     - U-Boot on the next boot (SM-08, SM-09)
     - Hold; forged tuple cannot match a durably reserved counter it did not know
   * - Context stale (counter from an earlier attempt)
     - Linux consumer via BCR comparison, then U-Boot (SM-09)
     - No mark or hold
   * - ``atomic_record_digest`` differs from the on-disk BCR
     - Linux consumer
     - No mark
   * - Context from a diagnostic boot reused in a later release boot
     - U-Boot (SM-18)
     - Ignored and cleared

Experiment E-01: GRUB preservation gate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

E-01 is an executable gate that must pass before B-04 implementation promotion. It has not run and no result is claimed.

Fixture: a U-Boot sandbox or ``qemu_arm64`` build carrying the future ``omarchy`` command in a diagnostic profile; a disk image with the fixed ESP layout, two BCR partitions, and one BSM partition; the pinned release GRUB image and generated configuration; a minimal kernel and initramfs whose only job is to dump ``/proc/device-tree/chosen/omarchy,boot-mode``, ``omarchy,boot-context``, and ``omarchy,boot-context-sha256`` and the two BCR copies to the console.

Procedure: boot through ``omarchy release`` with a PENDING ``slot-a``; capture the U-Boot-side canonical BootContext bytes and digest before launch; capture the Linux-side bytes; compare bytewise; repeat with ``omarchy try slot-a`` and require the two context properties to be absent and ``boot-mode`` to be ``diagnostic-untrusted``; repeat with a configuration that deliberately invokes ``devicetree`` and require the Linux consumer to refuse.

Pass criteria: bytewise equality in the release run, absence in the diagnostic run, and refusal in the mutated run, with the console logs and the disk image digests attached as evidence. Future path: ``test/py/tests/test_omarchy_boot_context.py`` (absent, NI-04).

Release command and configuration profile
-----------------------------------------

Command surface
~~~~~~~~~~~~~~~

Future path ``cmd/omarchy.c`` with the state machine in ``boot/omarchy_slot.c`` (both absent, NI-02 and NI-03). The command is ``omarchy`` with exactly these subcommands:

.. list-table::
   :header-rows: 1
   :widths: 24 16 60

   * - Subcommand
     - Writes BCR
     - Behavior
   * - ``omarchy release``
     - Yes (T-02 through T-09, T-11)
     - The only autoboot entry; the sequence in `Sequence of the release command`_; terminates only by launching or by HALT
   * - ``omarchy status``
     - No
     - Prints both copies' validity, sequences, states, digests, attempts, the last BSM header, and the last rejection string
   * - ``omarchy try <slot-a|slot-b|recovery>``
     - No
     - Manual diagnostic boot; performs the same manifest, board, and artifact verification; installs ``boot-mode = diagnostic-untrusted``; prints an untrusted banner; cannot affect any state
   * - ``omarchy recovery``
     - No
     - Equivalent to ``omarchy try recovery``
   * - ``omarchy halt``
     - No
     - Stops with the diagnostic census displayed

HALT behavior is deterministic: print the local code string, both copies' census, the slot that was attempted, and the reason; then drop to the console with the banner ``OMARCHY RELEASE BOOT HALTED: manual actions are untrusted``. The console in the release profile offers only ``omarchy``, ``reset``, and ``help``.

Kconfig symbols
~~~~~~~~~~~~~~~

Future symbols (absent, NI-02):

.. list-table::
   :header-rows: 1
   :widths: 34 66

   * - Symbol
     - Meaning
   * - ``CONFIG_OMARCHY_BOOT``
     - Enables the slot state machine, BCR reader and writer, BSM consumer, and BootContext installer
   * - ``CONFIG_CMD_OMARCHY``
     - Enables the ``omarchy`` command
   * - ``CONFIG_OMARCHY_BOOT_RELEASE``
     - Selects the release profile; depends on every exclusion in the next table being satisfied and fails the build otherwise
   * - ``CONFIG_OMARCHY_BCR_AUTH_HMAC``
     - Requires ``auth_algorithm = 2`` for any record expressing acceptance; selected by the release profile
   * - ``CONFIG_OMARCHY_RELEASE_CHANNEL``
     - String, ``stable`` in the release profile; edge and RC builds are distinct binaries with distinct values
   * - ``CONFIG_OMARCHY_BOOT_DIAG``
     - Enables ``omarchy try`` and ``omarchy recovery``; permitted in the release profile because both are structurally untrusted

Release profile
~~~~~~~~~~~~~~~

Future defconfig ``configs/apple_omarchy_release_defconfig`` (absent, NI-01) and default environment text ``board/apple/mac/mac-omarchy-release.env`` (absent). The resolved ``.config`` is recorded as a ``config-file/v1`` manifest input. The profile is an explicit, complete decision for each item below; a Kconfig default is never relied upon.

.. list-table::
   :header-rows: 1
   :widths: 40 12 48

   * - Symbol
     - Value
     - Reason
   * - ``CONFIG_BOOTCOMMAND``
     - ``omarchy release``
     - The only autoboot entry
   * - ``CONFIG_USE_BOOTCOMMAND``
     - ``y``
     - Required for the value above
   * - ``CONFIG_BOOTDELAY``
     - ``0``
     - No idle wait; console entry only through the keyed stop string or HALT
   * - ``CONFIG_AUTOBOOT_KEYED``, ``CONFIG_AUTOBOOT_STOP_STR_ENABLE``
     - ``y``
     - Physical-presence stop string; secrecy is not claimed and the console is untrusted by construction
   * - ``CONFIG_USE_PREBOOT``
     - ``y`` with ``preboot=usb start``
     - USB keyboards for manual recovery only
   * - ``CONFIG_USB_STORAGE``, ``CONFIG_CMD_USB_MASS_STORAGE``
     - ``n``
     - No USB block devices exist in the release image
   * - ``CONFIG_NO_NET``
     - ``y``
     - Already the Apple baseline; no network boot method can exist
   * - ``CONFIG_BOOTSTD``
     - ``n``
     - Removes every bootdev, bootmeth, bootflow, ``bootflow scan``, extlinux, PXE, script, and distro method
   * - ``CONFIG_EFI_BOOTMGR``, ``CONFIG_BOOTMETH_EFI_BOOTMGR``, ``CONFIG_CMD_BOOTEFI_BOOTMGR``
     - ``n``
     - ``BootNext``, ``BootOrder``, and ``Boot####`` cannot select anything
   * - ``CONFIG_CMD_BOOTEFI``, ``CONFIG_CMD_BOOTEFI_BINARY``
     - ``n``
     - No command can launch an arbitrary EFI file; ``omarchy release`` calls ``efi_binary_run()`` directly
   * - ``CONFIG_EFI_BINARY_EXEC``
     - ``y``
     - Required by ``efi_binary_run()``
   * - ``CONFIG_CMD_BOOTI``, ``CONFIG_CMD_BOOTM``, ``CONFIG_CMD_BOOTZ``, ``CONFIG_CMD_GO``
     - ``n``
     - Direct kernel launch is structurally unavailable
   * - ``CONFIG_EFI_VARIABLE_NO_STORE``
     - ``y``
     - No ``/ubootefi.var``; ``EFI_VARIABLE_FILE_STORE`` and ``EFI_RT_VOLATILE_STORE`` are ``n``; runtime ``SetVariable()`` is unsupported and irrelevant to slot state
   * - ``CONFIG_EFI_CAPSULE_ON_DISK``, ``CONFIG_EFI_HTTP_BOOT``, ``CONFIG_CMD_NVEDIT_EFI``
     - ``n``
     - No capsule, network, or variable editing path
   * - ``CONFIG_ENV_IS_NOWHERE``
     - ``y``
     - No persistent environment; ``ENV_IS_IN_FAT`` is ``n``; the default environment is compiled from the release env text
   * - ``CONFIG_ENV_WRITEABLE_LIST``, ``CONFIG_ENV_ACCESS_IGNORE_FORCE``
     - ``y``
     - No variable is writable at runtime, including ``bootcmd`` and ``bootargs``
   * - ``CONFIG_CMD_SAVEENV``, ``CONFIG_CMD_IMPORTENV``, ``CONFIG_CMD_EXPORTENV``, ``CONFIG_CMD_EDITENV``, ``CONFIG_CMD_SOURCE``, ``CONFIG_CMD_RUN``
     - ``n``
     - No mutable environment or script execution
   * - ``CONFIG_CMD_NVME``, ``CONFIG_CMD_GPT``, ``CONFIG_CMD_PART``, ``CONFIG_CMD_FAT``, ``CONFIG_CMD_EXT4``, ``CONFIG_CMD_LOADB``, ``CONFIG_CMD_LOADS``, ``CONFIG_CMD_LOADM``
     - ``n``
     - No raw block, partition, filesystem, or serial load commands from the console
   * - ``CONFIG_FWU_MULTI_BANK_UPDATE``, ``CONFIG_BOOTCOUNT_LIMIT``
     - ``n``
     - The generic FWU trial counter and the environment bootcount are not the Omarchy state machine
   * - ``CONFIG_NVME_APPLE``, ``CONFIG_OF_UPSTREAM_BUILD_VENDOR``, keyboard and video options
     - as ``apple_m1_defconfig``
     - Hardware baseline unchanged
   * - ``CONFIG_SHA256``, ``CONFIG_CRC32C``
     - ``y``
     - Required by the BCR tag and CRC
   * - ``CONFIG_OMARCHY_BOOT``, ``CONFIG_CMD_OMARCHY``, ``CONFIG_OMARCHY_BOOT_RELEASE``, ``CONFIG_OMARCHY_BCR_AUTH_HMAC``, ``CONFIG_OMARCHY_BOOT_DIAG``
     - ``y``
     - Release state machine with untrusted diagnostics

Enabled boot devices in release: the Apple NVMe namespace holding the ESP, resolved by UUID. Enabled boot methods: none other than ``omarchy release``. Recovery path: the PINNED recovery slot, verified against the same manifest rules. Console policy: reachable only through the keyed stop string or HALT, and every console action is labeled untrusted. Deterministic failure behavior: every failure ends in exactly one of launch-of-a-verified-slot, launch-of-verified-recovery, or HALT with a printed code; there is no path from any failure to a generic scan, removable medium, network source, alternate configuration file, or direct kernel image, and no diagnostic action can write PENDING, ACCEPTED, FAILED, or a last-known-good change.

Provenance and artifact roles
-----------------------------

PROVISIONAL, pending coordinator approval. Every role below is bound to a canonical manifest identity or to a named BLOCKED input. No illustrative local file names remain.

.. list-table::
   :header-rows: 1
   :widths: 24 46 30

   * - Role
     - Canonical binding
     - Producer and residual
   * - Immutable source
     - ``$.payload.components.boot_stack.source`` with ``source_kind = git-repository/v1``, ``repository_id``, ``source_commit`` (downstream tip), ``upstream_commit`` (Asahi base ``b33034a515a068381c4aa28831fb709e91300cd1`` for this branch), ``source_digest``, ``provenance_report_digest``; branches, tags, and refs are not identities
     - F-04 builder; whether ``source_digest`` is the tree object or archive digest is BLK-18
   * - Configuration lock
     - ``config_inputs[]`` with one ``defconfig/v1`` entry for ``configs/apple_omarchy_release_defconfig`` and one ``config-file/v1`` entry for the resolved ``.config``, each with ``source_digest``, ``normalized_content_digest``, ``policy_digest``
     - B-03 defines the inputs; F-04 records them
   * - Toolchain
     - ``toolchain_lock.entries[]`` for the AArch64 compiler, binutils, device-tree compiler, and host container, each with ``toolchain_digest`` and ``flags_digest``
     - Toolchain IDs and container are BLK-16
   * - Ordered patch queue
     - ``patch_lock.entries[]`` with ``order`` covering every downstream commit between ``upstream_commit`` and ``source_commit``
     - F-04 with the u-boot-omarchy maintainers
   * - Generated configuration and build report
     - ``report_lock.entries[]`` with ``report_kind = build/v1`` for the two-builder comparison, ``lint/v1`` for static analysis, ``compatibility/v1`` for the DT and predecessor interface report
     - F-04 and B-03
   * - U-Boot binary
     - ``components.boot_stack.artifacts[]`` entry with ``kind = boot-image/v1`` for ``u-boot-nodtb.bin`` with ``size_bytes`` and ``content_digest``
     - Artifact ID and media type are BLK-09
   * - DT artifacts
     - ``components.dtb_set.artifacts[]`` with ``kind = dtb/v1`` and ``components.dtb_set.dt_schema``; U-Boot verifies the DTB files GRUB may load and consumes the working DT from the predecessor
     - K-01 and the human predecessor owner
   * - Firmware bundle
     - ``components.firmware_bundle.artifacts[]`` and ``firmware_schema``; cross-checked to the registry ``firmware`` record
     - K-01; U-Boot does not load firmware
   * - GRUB image and configuration
     - Two ``components.boot_stack.artifacts[]`` entries with ``kind = boot-image/v1`` distinguished by ``media_type``
     - IDs and media types BLK-09; configuration generated by F-05
   * - Signatures
     - Envelope ``signatures[]`` under ``manifest-release``; per-artifact ``signature_policy_id``
     - F-03 keys and policy
   * - SBOM and provenance
     - ``components.boot_stack.provenance`` with ``source_observation_digest``, ``build_input_digest``, ``attestation_digest``, ``producer_binding_digest``; license and notice inventory per F-06 for U-Boot (GPL-2.0-or-later) and GRUB (GPL-3.0-or-later with source offer)
     - F-04 and F-06
   * - Candidate and manifest identity
     - ``$.payload.document_id``, envelope ``payload_digest``, ``$.payload.channel``, ``$.payload.release_version``
     - F-05 assembles; F-07 promotes
   * - Rollback predecessor
     - ``components.boot_stack.rollback.previous_manifest_ids`` and ``previous_component_ids``; top-level ``rollback`` projection with ``failure_attempt_limit`` and ``minimum_retention``
     - F-05; enforced by T-03
   * - Opaque predecessor envelope
     - No manifest path exists in the F-02 candidate for the human-signed stage-1/stage-2 artifact identity, interface attestation, license inventory, and provenance
     - BLK-10; the assembled predecessor payload that embeds ``u-boot-nodtb.bin`` is an F-05 assembly artifact whose role is also BLK-10

Dependency and handoff matrix
-----------------------------

Each row names the producer, the consumer, the artifact and how its digest is bound, the freshness rule, the scope, what happens on rejection, who owns the residual, and the gate the handoff must precede.

.. list-table::
   :header-rows: 1
   :widths: 7 9 9 22 12 11 14 9 7

   * - ID
     - Producer
     - Consumer
     - Artifact and digest binding
     - Freshness
     - Scope
     - Rejection
     - Residual owner
     - Due before
   * - DEP-01
     - F-02
     - B-03
     - Ratified schema set; ``schema_set_digest`` compiled into the boot binding
     - Exact digest equality
     - Eight payload types, boot limits
     - ``BINDING_INTEGRITY_FAILURE``; every object absent
     - F-02 owner
     - B-03 implementation admission
   * - DEP-02
     - F-02
     - B-03
     - Generated boot binding for U-Boot; ``generated-output.lock`` entry with ``output_role = boot-binding`` (BLK-04)
     - Compiled lock equality
     - Parse, canonicalize, verify for the three boot-consumed types
     - Build fails
     - F-02 implementation owner
     - B-03 implementation admission
   * - DEP-03
     - F-03
     - B-03, B-04
     - ``Trusted<TrustContext>`` bundle with ``manifest-release`` and ``boot-runtime`` bindings embedded in the U-Boot image (BLK-05)
     - ``revocation_epoch`` and binding expiry under BLK-06
     - Two roles only
     - ``TRUST_FAILURE``; no slot verifies; recovery or HALT
     - F-03
     - B-04 release profile gate
   * - DEP-04
     - F-03 with human predecessor owner
     - B-04
     - ``OMARCHY_BCR_AUTH_KEY`` provisioning (BLK-03)
     - Per-device
     - BCR MAC only
     - Release profile cannot accept any record
     - F-03
     - B-04 release profile gate
   * - DEP-05
     - F-04
     - B-03
     - Builder definitions, toolchain lock, two-builder comparison, provenance attestations for ``boot-stack``
     - Per candidate
     - U-Boot and GRUB artifacts
     - Candidate rejected at assembly
     - F-04
     - F-05 assembly
   * - DEP-06
     - F-05
     - B-04
     - Immutable ``platform-manifest/v1`` per slot, generated GRUB configuration, hostile cross-repository fixtures, artifact IDs (BLK-09)
     - ``payload_digest`` equality at every boot
     - One board target set
     - T-08 or T-03 refusal
     - F-05
     - B-04 implementation admission
   * - DEP-07
     - F-06
     - B-03
     - License, notice, and source-offer inventory for U-Boot and GRUB artifacts
     - Per candidate
     - ``boot-stack``
     - Candidate rejected at assembly
     - F-06
     - F-05 assembly
   * - DEP-08
     - B-04
     - F-07
     - Passing sandbox suite, E-01 evidence, hostile fixture census, release profile ``.config``
     - Per candidate
     - Stable promotion prerequisite
     - Promotion refused
     - B-04
     - F-07 promotion
   * - DEP-09
     - Human owner (B-01, B-02, B-05 through B-08)
     - B-03, B-04
     - Signed opaque predecessor envelope with interface attestation for DT address, root compatible, ``/chosen`` properties, and memory map (BLK-10)
     - Per cohort
     - U-Boot entry inputs only
     - Board fails admission; no compatibility claim
     - Human predecessor owner
     - Q-04 through Q-08
   * - DEP-10
     - B-04
     - K-01
     - BootContext property specification and BCR authenticated-region digest rule
     - Version-locked with ``format_major``
     - Linux consumer adapter
     - ``TRUST_BOUNDARY_FAILURE``; no mark
     - B-04
     - K-02
   * - DEP-11
     - K-01, K-02
     - B-04
     - Kernel image, initramfs, and DTB artifacts with digests; the OS boot-health writer and mark writer
     - ``payload_digest`` equality
     - Slot payload
     - T-08; no mark
     - K-01
     - B-04 implementation admission
   * - DEP-12
     - B-04
     - P-05
     - Slot layout, staging discipline (manifest written last, active slot never overwritten), BSM writer contract
     - ``format_major``
     - Update UX
     - Staged slot never becomes PENDING
     - P-05
     - P-05
   * - DEP-13
     - B-04
     - P-04
     - ``omarchy status`` projection fields for diagnostics with privacy bounds
     - Per release
     - Diagnostics only
     - Field omitted
     - P-04
     - P-04
   * - DEP-14
     - I-03, I-04, I-05
     - B-03, B-04
     - Approved plan naming the two BCR partitions and BSM partition as targets; T-01 provisioning; ``control_set_id`` (BLK-15); assembled predecessor payload installation
     - Per install
     - Omarchy-owned partitions only
     - Install fails closed before mutation
     - I-04
     - I-09
   * - DEP-15
     - Q-00, Q-01
     - B-03
     - Board registry rows with ``identity_match.linux`` for every board U-Boot may boot (BLK-11)
     - ``board_registry_digest`` equality
     - Board admission
     - Code 18; HALT
     - Q-00
     - B-03 implementation admission
   * - DEP-16
     - B-04
     - Q-04 through Q-08
     - Physical qualification rows: power-loss cut points, boot loops, ten update and rollback cycles per unit, recovery boot, DFU rehearsal
     - Per board and firmware baseline
     - Physical evidence only
     - Board not promoted
     - Hardware lab
     - F-07 promotion
   * - DEP-17
     - G-01 through G-05
     - none
     - No boot interface between Mesa and U-Boot; listed to make the absence explicit
     - not applicable
     - none
     - not applicable
     - none
     - not applicable
   * - DEP-18
     - F-02
     - B-04
     - Verified clock policy for boot consumers (BLK-06), ``DocumentId`` grammar (BLK-07), lineage allocation (BLK-08), ``boot_artifact_digest`` semantics (BLK-12), core envelope bound (BLK-13), fixed ``ExpectedContext`` IDs (BLK-14)
     - Ratification
     - Boot rows
     - Rows SM-21 and the affected fields stay BLOCKED
     - F-02
     - B-04 implementation admission

Hostile fixtures
----------------

Every fixture is a single mutation against an otherwise accepted state. The expected result is the code, the decision, and the component that must reject. These fixtures are required test content; none has run (NI-05).

.. list-table::
   :header-rows: 1
   :widths: 7 25 38 30

   * - ID
     - Fixture
     - Mutation
     - Expected result
   * - HF-01
     - ``envelope-replay``
     - Present a previously accepted mark envelope byte-for-byte on a later boot
     - SM-09 ``BOOT_COUNTER_FAILURE``; U-Boot; hold
   * - HF-02
     - ``envelope-transplant``
     - Move a valid mark from another board's control set into this BSM
     - SM-08 or SM-19; U-Boot; hold
   * - HF-03
     - ``typed-digest-substitution``
     - Replace ``manifest_digest`` in the mark with a ``document_id`` digest or a different valid manifest's digest
     - SM-08 ``BOOT_CONTEXT_MISMATCH``; U-Boot
   * - HF-04
     - ``wrong-board``
     - Mark ``board_id`` for a sibling board
     - SM-08; U-Boot
   * - HF-05
     - ``wrong-manifest``
     - Mark bound to the last-known-good manifest while the pending slot carries a new one
     - SM-08; U-Boot
   * - HF-06
     - ``wrong-slot``
     - Mark ``slot_id = slot-a`` while ``slot-b`` is pending
     - SM-08; U-Boot
   * - HF-07
     - ``wrong-generation``
     - Mark ``slot_generation`` one below the descriptor
     - SM-08; U-Boot
   * - HF-08
     - ``wrong-lineage``
     - Mark ``lineage_id`` from the tombstoned lineage
     - SM-08; U-Boot
   * - HF-09
     - ``wrong-source-generation``
     - Mark ``source_generation`` reset to 0
     - SM-08; U-Boot
   * - HF-10
     - ``forged-mark-signature``
     - Flip one signature byte
     - SM-07 ``BOOT_MARKER_AUTH_FAILURE``; U-Boot
   * - HF-11
     - ``forged-mark-role``
     - ``signer_role = manifest-release`` on a mark
     - SM-07; U-Boot
   * - HF-12
     - ``missing-mark``
     - Container CONSUMED or absent after a pending boot
     - SM-03 or SM-01; budget continues; T-06 at the limit
   * - HF-13
     - ``replayed-mark-generation``
     - ``marker_generation`` equal to BCR-S12
     - SM-10 ``EXPIRY_OR_REPLAY_FAILURE``; U-Boot
   * - HF-14
     - ``counter-reset``
     - BCR copy with ``attempt_counter`` lower than the other valid copy at a higher sequence
     - State consistency failure; copy invalid; C2-05
   * - HF-15
     - ``counter-wrap``
     - Descriptor ``attempt_counter`` at the maximum before T-04
     - ``BOOT_COUNTER_FAILURE``; no launch of that slot; T-06 path
   * - HF-16
     - ``both-copies-corrupt``
     - Random bytes in both partitions
     - C2-07; no write; recovery or HALT
   * - HF-17
     - ``torn-write-each-phase``
     - Cut power at each of the eleven listed points
     - The listed C2 or SM case, verified by the captured copies
   * - HF-18
     - ``stale-copy-selection``
     - Copy 1 valid at sequence n, copy 0 valid at sequence n+1 with a different selected slot
     - C2-03 selects copy 0; copy 1 repaired
   * - HF-19
     - ``concurrent-writer``
     - Re-enter the writer from a diagnostic subcommand or a second call
     - Code 17; no write
   * - HF-20
     - ``power-loss-before-flush``
     - Cut after the write completes but before the flush returns
     - Torn or missing copy per C2-11 and C2-12; never a half-committed launch
   * - HF-21
     - ``power-loss-after-flush``
     - Cut after the flush and read-back
     - Committed state observed on the next boot
   * - HF-22
     - ``malicious-removable-media``
     - USB disk with ``/EFI/BOOT/bootaa64.efi`` and an extlinux configuration attached
     - Not enumerated; no bootdev exists; release sequence unchanged
   * - HF-23
     - ``network-media``
     - Network boot server on the link
     - No network stack; no effect
   * - HF-24
     - ``mutable-environment``
     - ``/ubootefi.var``, a FAT environment file, and ``BootOrder`` pointing at another slot on the ESP
     - Not read; ``ENV_IS_NOWHERE`` and ``EFI_VARIABLE_NO_STORE``; no effect
   * - HF-25
     - ``grub-context-stripping``
     - GRUB configuration invoking ``devicetree`` with a DT lacking the properties
     - Linux consumer refuses; no mark; E-01 mutated run
   * - HF-26
     - ``grub-context-mutation``
     - Modify one byte of ``omarchy,boot-context`` before the kernel
     - Linux consumer SHA-256 mismatch; no mark
   * - HF-27
     - ``direct-kernel-bypass``
     - Attempt ``booti`` or ``bootefi`` from the console, or a kernel EFI stub placed as the GRUB artifact
     - Commands absent; artifact digest mismatch T-08
   * - HF-28
     - ``rollback-recursion``
     - After T-06, stage the identical manifest again in the FAILED slot
     - T-03 refused because ``payload_digest`` equals the FAILED descriptor
   * - HF-29
     - ``missing-last-known-good``
     - PENDING slot exhausts its budget with ``last_known_good_slot = 0``
     - T-06 selects recovery; never an unverified slot
   * - HF-30
     - ``alternate-stable-writer``
     - A manifest with ``channel = stable`` signed by a valid key whose binding lacks the ``manifest-release`` role, or a manifest not produced by F-07 promotion
     - ``SIGNATURE_CONTEXT_MISMATCH`` or ``TRUST_FAILURE``; slot never PENDING; F-07 remains the only stable writer
   * - HF-31
     - ``unkeyed-accepted-record``
     - Unkeyed record claiming ACCEPTED with a non-zero last-known-good slot
     - Copy invalid; C2-05 or C2-07; the OS cannot express acceptance
   * - HF-32
     - ``lkg-overwritten-in-place``
     - Replace the last-known-good slot's manifest with a different signed manifest
     - T-09 code 19; recovery; the slot is never re-trialed silently
   * - HF-33
     - ``recovery-pin-mismatch``
     - Recovery slot manifest differs from the PINNED digest
     - Recovery refused; HALT

Empirical residuals
-------------------

These are NOT IMPLEMENTED gates observed at the current tip. They are never design accomplishments. Each has an owner and a due-before condition.

.. list-table::
   :header-rows: 1
   :widths: 7 48 15 30

   * - ID
     - Observation
     - Owner
     - Due before
   * - NI-01
     - ``configs/apple_m1_defconfig`` still uses ``CONFIG_BOOTCOMMAND="bootflow scan -b"``; ``configs/apple_omarchy_release_defconfig`` does not exist
     - B-03
     - B-03 implementation admission
   * - NI-02
     - No Omarchy command, Kconfig symbol, schema, binding, state writer, or validator exists in this tree; ``cmd/omarchy.c``, ``boot/omarchy_slot.c``, ``schemas/platform-manifest.json``, and ``schemas/boot-health.json`` are absent, and the last two must never exist here because schemas are published only by ``omarchy-apple-platform``
     - B-04
     - B-04 implementation admission
   * - NI-03
     - No unit or Python test for any Omarchy contract exists; ``test/dm/test_omarchy_slot.c`` and ``test/py/tests/test_omarchy_slots.py`` are absent
     - B-04
     - B-04 implementation admission
   * - NI-04
     - The success-mark and GRUB preservation experiment E-01 has not run; ``test/py/tests/test_omarchy_boot_context.py`` is absent
     - B-04
     - B-04 implementation promotion
   * - NI-05
     - No hostile fixture in `Hostile fixtures`_ has been executed
     - B-04
     - B-04 implementation promotion
   * - NI-06
     - Local configuration and build validation was unavailable: host GNU Make 3.81 fails at ``Makefile:247``; no ``.config`` was resolved and no binary was produced
     - B-03
     - B-03 implementation admission on a pinned builder
   * - NI-07
     - No Ed25519 verifier and no JSON or RFC 8785 canonicalizer exists under ``lib/``; the boot binding must supply them within the bounded limits
     - B-04 with DEP-02
     - B-04 implementation admission
   * - NI-08
     - The block layer has no flush operation and the NVMe driver never issues ``nvme_cmd_flush``; the commit procedure's flush steps cannot be honored until a flush operation exists and is proven on Apple NVMe after power removal
     - B-04
     - B-04 implementation admission and Q-04 power-loss rows
   * - NI-09
     - No verified clock source exists in U-Boot on Apple hardware; SM-21 is governed by BLK-06
     - F-02 and F-03
     - B-04 implementation admission
   * - NI-10
     - No physical qualification evidence exists for any board; nothing in this document is a support or compatibility claim
     - Hardware lab
     - Q-04 through Q-08
   * - NI-11
     - No exact release configuration record, two-builder comparison, SBOM, or provenance exists for ``boot-stack``
     - F-04 and B-03
     - F-05 assembly
   * - NI-12
     - ``ENV_IS_IN_FAT``, ``EFI_VARIABLE_FILE_STORE``, ``EFI_BOOTMGR``, ``BOOTSTD``, and the EFI bootmeths remain enabled by default for the Apple baseline; the release exclusions are design only
     - B-03
     - B-03 implementation admission
   * - NI-13
     - ``asahi_esp_devpart()`` falls back to the first EFI system partition when the UUID is absent or unmatched; the release rule of UUID-only resolution is not implemented
     - B-03
     - B-03 implementation admission
   * - NI-14
     - The T-01 provisioning writer, the OS BSM writer, and the Linux BootContext consumer do not exist in any repository
     - I-04, K-01, P-05
     - B-04 implementation promotion

BLOCKED external constants
--------------------------

Each constant is owned outside this lane. Until ratified, U-Boot has no value for it, cannot be built for release, and B-03/B-04 admission is BLOCKED. No value is guessed here.

.. list-table::
   :header-rows: 1
   :widths: 8 40 18 34

   * - ID
     - Constant
     - Owner
     - Due before
   * - BLK-01
     - ``OMARCHY_BCR_PARTITION_TYPE_GUID`` for the two boot-control partitions in the typed storage registry
     - F-02 with I-03
     - B-04 implementation admission
   * - BLK-02
     - ``OMARCHY_BSM_PARTITION_TYPE_GUID`` for the success-mark partition
     - F-02 with I-03
     - B-04 implementation admission
   * - BLK-03
     - ``OMARCHY_BCR_AUTH_KEY`` source, provisioning, and rotation; the key must be unavailable to the Linux OS at runtime, and if F-03 rules that no such source exists on Apple hardware the residual becomes an owner-accepted threat-model entry rather than a silent downgrade to algorithm 1
     - F-03 with the human predecessor owner
     - B-04 release profile gate
   * - BLK-04
     - Boot binding target for U-Boot: the candidate names ``rust-boot`` outputs with ``output_role = boot-binding`` while U-Boot is a C code base; the ratified language, linkage, and bounded-memory report for the U-Boot consumer
     - F-02
     - B-03 implementation admission
   * - BLK-05
     - Embedded ``Trusted<TrustContext>`` bundle format, offline expiry handling, and rotation procedure for a firmware consumer without network
     - F-03
     - B-04 release profile gate
   * - BLK-06
     - Verified clock policy for boot consumers; U-Boot proposes counter-based freshness with a not-before floor from BCR-20 and BSM-12 for ratification
     - F-02 and F-03
     - B-04 implementation admission
   * - BLK-07
     - ``DocumentId`` grammar; BCR-S09 stores its SHA-256 so the record width is independent of the ruling
     - F-02
     - B-04 implementation admission
   * - BLK-08
     - ``lineage_id`` allocation rule; U-Boot proposes a deterministic derivation from board, manifest digest, slot, and slot generation because U-Boot has no attested random source
     - F-02
     - B-04 implementation admission
   * - BLK-09
     - Artifact IDs and media types for the U-Boot image, GRUB image, GRUB configuration, kernel image, initramfs, DTB set, and recovery payload
     - F-05 with F-04
     - B-04 implementation admission
   * - BLK-10
     - Manifest path and interface attestation schema for the opaque predecessor envelope, and the role of the assembled predecessor payload that embeds ``u-boot-nodtb.bin``
     - F-02 with the human predecessor owner (B-01)
     - B-03 implementation admission
   * - BLK-11
     - Delivery form of the ``board-registry/v1`` document to U-Boot (embedded at build or staged in the slot) and the exact ``identity_match.linux`` matching rule
     - F-02 with Q-00
     - B-03 implementation admission
   * - BLK-12
     - Semantics of ``slot.boot_artifact_digest``; U-Boot proposes the GRUB image ``content_digest``
     - F-02
     - B-04 implementation admission
   * - BLK-13
     - Complete ``boot-health/v1`` core envelope byte maximum; U-Boot reserves 4,096 bytes
     - F-02
     - B-04 implementation admission
   * - BLK-14
     - Fixed ``project_id``, ``repository_id``, and ``slice_id`` values in ``ExpectedContext`` for the ``boot-health-core`` and ``boot-success-marker`` rows
     - F-02 with F-03
     - B-04 implementation admission
   * - BLK-15
     - ``control_set_id`` derivation from the installer plan target identity
     - I-03
     - B-04 implementation admission
   * - BLK-16
     - Toolchain IDs, versions, and builder container digest for ``boot-stack``
     - F-04
     - F-05 assembly
   * - BLK-17
     - Recovery slot manifest policy: channel, signer, retention, and how the PINNED digest is refreshed
     - F-05 with F-07
     - B-04 implementation admission
   * - BLK-18
     - Whether ``source_digest`` is the git tree object digest or a source archive digest
     - F-02 with F-04
     - F-05 assembly

Acceptance and power-loss test plan
-----------------------------------

The first implementation extends the existing surfaces rather than claiming they cover the contract: ``test/dm/fwu_mdata.c`` for the pattern of two-copy metadata tests, ``test/py/tests/test_gpt.py`` for identity-tuple resolution and rejection of an ambiguous layout, ``test/boot/bootflow.c`` only to prove that no bootflow exists in the release profile, ``test/py/tests/test_efi_bootmgr.py`` only to prove that ``BootOrder`` cannot select anything, and ``test/py/tests/test_distro.py`` as a console-interaction reference. New sandbox tests use a disposable host-bound disk image and inject a failure at every durable write boundary; after every simulated reset the harness captures both BCR copies, the BSM header, the slot directory listing, the selected slot, the attempt counter, and the decision string.

The B-04 implementation gate is: every C2, SM, T, and HF row has a deterministic sandbox result; E-01 passes; the release ``.config`` resolves on a pinned builder with the two-builder comparison; and every BLK constant is ratified. The physical gate additionally requires the Q-04 rows on disposable qualified hardware with rehearsed outer recovery. A green sandbox run, a U-Boot prompt, a recognized SoC, or a booting desktop is never qualification evidence.

Closing statement
-----------------

This document is design only. It contains no placeholder and no unresolved design question: every open item is a named BLOCKED constant with an owner and a due-before gate, or a NOT IMPLEMENTED residual with an owner and a due-before gate. F-02 remains REJECTED, B-03 and B-04 remain open and not started, their implementation admission remains BLOCKED, no build succeeded, no experiment ran, no hardware booted, and nothing here is DONE.
