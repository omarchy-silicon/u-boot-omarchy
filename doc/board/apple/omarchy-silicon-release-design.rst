.. SPDX-License-Identifier: GPL-2.0+

Omarchy Silicon release and boot-slot design
============================================

Status and scope
----------------

This is the B-03/B-04 design note for ``omarchy-silicon/u-boot-omarchy``, correction round 2. It is a design-only document. It is not an implementation, a build result, a schema, a binding, a test result, a compatibility claim, a support claim, a qualification record, a release approval, or a recovery claim. Nothing described here is DONE. B-03 and B-04 remain open, not-started slices in the canonical program ledger, and their implementation admission is BLOCKED until the upstream contracts named in this document are ratified.

The canonical program is ``omarchy-apple-platform/PROGRAM.md`` at the ratified commit ``58302d148f0e8b855578f9aa518ff1c5eb48c515``. This document binds U-Boot to the frozen cross-repository contracts listed in that program and creates no second authority.

Every wire detail derived from the F-02 candidate ``docs/design/platform-schema.md`` at ``c315c7e79928d0041deb582bed79a61074361b21`` is PROVISIONAL. That candidate is REJECTED and frozen pending the owner checkpoint recorded in the program. Provisional details are quoted so that the U-Boot binding is exact rather than illustrative; they are not a local shadow schema, they are not implemented here, and they do not become authority by appearing in this document. Where an F-02- or F-03-owned identifier is not ratified, this document uses a named BLOCKED constant from the table in `BLOCKED external constants`_ instead of a guessed value.

The coordinator has fenced ``m1n1-omarchy`` and every m1n1 path as an opaque human-produced boundary. This document does not inspect, read, analyze, edit, test, clone, fetch, browse, traverse, characterize, or make claims about that repository or its outputs. The predecessor stage appears here only as a coordinator-supplied signed opaque artifact envelope and as the observable inputs U-Boot consumes at its own entry point.

Ratification and fail-closed gate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The current text is a design proposal only. The F-02 and F-03 documents, generated bindings, authority bindings, monotonic-floor authority, device/installation identity authority, and their digests are external inputs, not values defined by this file. Before any ``Trusted<T>`` constructor, slot read, recovery launch, state transition, or release build, U-Boot must compare the locally compiled contract identity with the exact ratified F-02/F-03 identities and verify the authenticated source records. Missing, expired, revoked, unavailable, or mismatched canonical dependencies have exactly one result: ``TRUST_BOUNDARY_FAILURE`` or ``BINDING_INTEGRITY_FAILURE``, no write, no selection, no recovery fallback, and ``HOLD/HALT``. A local alias, guessed value, stable-channel signature, copied disk value, or opaque predecessor output cannot satisfy this gate.

The proposal becomes eligible for implementation only after the coordinator ratifies the exact record, manifest, profile, promotion, storage, and authority relations below. Ratification does not imply implementation, CI, power-cut evidence, physical qualification, release approval, or DONE status.

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

PROVISIONAL, pending coordinator approval. Every authenticated object is carried by the closed ``omarchy-signed/v1`` envelope with exactly ``format``, ``payload_type``, ``payload_version``, ``domain``, ``context``, ``schema_set_digest``, ``payload``, and ``signatures``. The payload carries the common fields ``schema``, ``schema_set_digest``, ``document_id``, ``issuer``, ``issued_at``, and ``expires_at``. Each signature entry carries exactly ``key_id``, ``signer_role``, ``algorithm = ed25519``, ``signature_format = raw-ed25519/v1``, and a 64-byte signature in unpadded base64url. Normatively, ``payload_digest`` is derived only as ``sha256(JCS(payload))``; it is not a wire or envelope property, is never parsed from JSON, and is the sole derived payload value stored or bound by U-Boot. The signed preimage is ``ASCII("omarchy-auth-preimage/v1") || 0x00 || JCS(A)`` where ``A`` contains the envelope format, signature format, key ID, signer role, algorithm, domain, context, payload type, payload version, schema-set digest, the derived payload digest, the ``anti_transplant`` object (document ID, schema, payload type, payload version, schema-set digest, domain, context), and the complete payload. The stable ``document_id`` is never a content digest.

U-Boot's consumer follows the only construction path: ``strict_parse -> canonicalize -> derive_payload_digest -> verify(Canonical<T>, Trusted<TrustContext>, VerifiedClock, ExpectedContext) -> generated F-02 validator -> Trusted<T>``. For ``platform-manifest/v1``, the generated manifest-authority validator and the F-07 promotion-receipt validator are mandatory predecessors of ``Trusted<PlatformManifest>``; no consumer-local alias can skip them. A failed step yields no partial value and no authority. ``schema_set_digest`` is compared byte-for-byte with the schema-set digest compiled into the U-Boot boot binding; any other value is ``BINDING_INTEGRITY_FAILURE`` and the boot consumer treats the object as absent.

Authority resolution uses only the closed ``AuthorityRoleBinding`` records inside the F-03-supplied ``Trusted<TrustContext>``: ``binding_schema``, ``authority_id``, ``role``, ``actor_id``, ``account_id``, ``key_ids``, ``allowed_methods``, ``service_policy_id``, ``service_policy_digest``, ``issued_at``, ``expires_at``, and ``binding_digest``. The ten roles are ``board-admission``, ``manifest-release``, ``installer-planner``, ``owner-authorization``, ``ci-conformance``, ``qualification-lab``, ``boot-runtime``, ``dtb-authority``, ``evidence-reader``, and ``f07-promotion``. U-Boot resolves exactly three roles at boot: ``manifest-release`` for the slot manifest, ``f07-promotion`` for the embedded promotion receipt, and ``boot-runtime`` for the boot-health core and the boot-success mark. A key ID or role string in an envelope is an authenticated claim to be checked against the binding, never a hint that grants anything.

``ExpectedContext`` is constructed by U-Boot for each verification with the exact row from the eight-row signing contract and the separately ratified F-07 receipt context: ``payload_type``, ``payload_version``, ``domain``, ``context``, ``project_id``, ``repository_id``, ``slice_id``, ``operation``, ``board_id``, ``manifest_id``, ``manifest_digest``, ``schema_set_digest``, ``target_account_id``, ``target_account_binding``, ``target_identity_digests``, and ``policy_digest``. The fixed ``project_id``, ``repository_id``, and ``slice_id`` values for the boot rows are BLK-14. A mismatch on any non-null member is ``SIGNATURE_CONTEXT_MISMATCH`` or ``BOOT_PROMOTION_FAILURE`` for the receipt.

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
     - ``$.payload.document_id`` and the derived ``sha256(JCS($.payload))``
     - Manifest identity bound into BCR-S09 and BCR-S08; no ``payload_digest`` JSON path exists
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
     - Complete profile: allowed classes, check IDs and exact order, measurement names/units/bounds, ``retry_limit``, ``failure_limit``, source evidence/record, ``profile_id``, ``profile_digest``, ``rollback_manifest_ids``, and freshness policy; source of BCR-S04 and ``Trusted<BootHealthCore>``
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
   * - ``platform-manifest/v1``
     - ``$.payload.artifacts``, ``$.payload.package_set``, ``$.payload.compatibility``, ``$.payload.firmware_schema``, ``$.payload.rollback`` and every authoritative ``$.payload.components[*]`` projection
     - Generated F-02 manifest-authority validator recomputes and compares exact length, key, order, and every field before ``Trusted<PlatformManifest>``; mismatch is ``BOOT_PROJECTION_FAILURE``
   * - ``platform-manifest/v1``
     - ``$.payload.promotion_receipt``
     - Exact F-07 receipt binding for candidate, rollback, closure, board/profile/qualification, legal/public-ledger, channel, manifest, generation, lineage, health, signer/context, freshness, and anti-replay; missing or invalid is ``BOOT_PROMOTION_FAILURE``
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
     - F-05 and F-07 bind the qualification record into the promotion receipt; U-Boot verifies the receipt's qualification digest and never treats channel or a manifest signature as qualification evidence

Manifest projection and promotion admission
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The generated F-02 manifest-authority validator is a mandatory predecessor to ``Trusted<PlatformManifest>``. It receives only a strictly parsed, canonical ``platform-manifest/v1`` envelope and the exact F-02 schema-set digest. It recomputes the authoritative component projections and compares, in this fixed order, ``artifacts``, ``package_set``, ``compatibility``, ``firmware_schema``, and ``rollback``. For every projection it compares array length, object key set, key order, array order, scalar type, scalar value, nested object, and every field. Extra, missing, duplicated, reordered, or differently typed properties are failures, not ignored extensions. The validator returns one deterministic result tuple ``(code, path, phase, result)`` for the first canonical path in that order; no later validator may replace it.

The only accepted manifest construction is ``strict_parse -> canonicalize -> derive_payload_digest -> envelope and signer verification -> generated F-02 projection validation -> F-07 promotion-receipt validation -> Trusted<PlatformManifest>``. The projection validator is not a best-effort warning and is not a consumer-local alias. If the generated validator, its schema-set digest, or its source record is missing or mismatched, the result is ``BINDING_INTEGRITY_FAILURE`` or ``BOOT_PROJECTION_FAILURE`` and the manifest is absent. Stable-channel signature verification without this predecessor never constructs the trusted type.

The F-07 promotion receipt is a separately authenticated, owner-authorized relation embedded at ``$.payload.promotion_receipt`` and bound into the canonical manifest. It is not a ninth top-level payload type or a consumer-local alias. Its exact future-ratified object must contain ``receipt_schema``, ``candidate_digest``, ``rollback_digest``, ``closure_digest``, the complete required-slice closure, ``board_id``, ``profile_digest``, ``qualification_record_digest``, ``legal_digest``, ``public_ledger_digest``, ``channel``, ``manifest_id``, ``manifest_digest``, ``generation``, ``lineage_id``, ``health_evidence_digest``, ``signer_id``, ``signing_context``, ``issued_at``, ``expires_at``, ``replay_id``, and ``receipt_signature``. U-Boot verifies every field, the receipt signer and context under ``f07-promotion``, freshness, anti-replay, and equality to the manifest and current record. Missing, stale, forked, incomplete, or mismatched receipt is ``BOOT_PROMOTION_FAILURE`` at ``manifest.admission.promotion_receipt`` with result ``REJECT/HALT``. A stable-channel signature alone is never sufficient.

.. list-table::
   :header-rows: 1
   :widths: 22 32 18 28

   * - Check order
     - Exact path or comparison
     - First-failure code
     - Phase and result
   * - 1
     - ``$.payload.artifacts`` against recomputed component artifact projection; exact length, key, order, and every field
     - ``BOOT_PROJECTION_FAILURE``
     - ``manifest.projection.artifacts``; ``REJECT/HALT``
   * - 2
     - ``$.payload.package_set`` against the recomputed package projection; exact length, key, order, and every field
     - ``BOOT_PROJECTION_FAILURE``
     - ``manifest.projection.package_set``; ``REJECT/HALT``
   * - 3
     - ``$.payload.compatibility`` against the recomputed compatibility projection; exact length, key, order, and every field
     - ``BOOT_PROJECTION_FAILURE``
     - ``manifest.projection.compatibility``; ``REJECT/HALT``
   * - 4
     - ``$.payload.firmware_schema`` against the recomputed firmware-schema projection; exact length, key, order, and every field
     - ``BOOT_PROJECTION_FAILURE``
     - ``manifest.projection.firmware_schema``; ``REJECT/HALT``
   * - 5
     - ``$.payload.rollback`` against the recomputed rollback projection; exact length, key, order, and every field
     - ``BOOT_PROJECTION_FAILURE``
     - ``manifest.projection.rollback``; ``REJECT/HALT``
   * - 6
     - ``$.payload.promotion_receipt`` complete relation and exact equality to manifest, record, board, profile, and qualification inputs
     - ``BOOT_PROMOTION_FAILURE``
     - ``manifest.admission.promotion_receipt``; ``REJECT/HALT``

Boot-health profile binding
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before ``Trusted<BootHealthCore>`` exists, U-Boot must bind the complete manifest-declared BootCheck profile. The bound profile is the exact allowed class set, check ID set and order, measurement name, unit, lower and upper bounds for every check, retry limit, failure limit, source evidence and source record identity, status vocabulary, profile digest, and freshness interval. The core's checks must have exactly the declared set and order, each ID exactly once, each class allowed, each measurement name and unit exact, each measurement within its declared bounds, each source record equal to the declared evidence, and each status equal to the closed vocabulary. ``checks_digest`` is recomputed only after those checks; it cannot launder an invalid profile.

The core path is ``strict_parse -> canonicalize -> verify boot-runtime -> compare profile identity and digest -> compare allowed classes and exact check set/order -> compare measurement names/units/bounds -> compare retry/failure limits -> compare source evidence/record -> verify freshness -> Trusted<BootHealthCore>``. Any failure returns ``BOOT_PROFILE_FAILURE`` or ``BOOT_REQUIRED_CHECK_FAILURE`` with the first failing canonical check path and no trusted core. Success-mark validation is a separate later path: ``Trusted<BootHealthCore> -> strict_parse/verify boot-success-mark -> exact core, pending descriptor, counter, freshness, replay, and rollback comparisons -> T-05``. A valid core never implies a valid mark, and a valid mark never constructs a core.

Freshness is checked against the ratified F-02/F-03 clock or monotonic policy at both core and mark admission. Until that authority is ratified, the result is ``BOOT_FRESHNESS_FAILURE`` and ``HOLD/HALT``; counter equality alone is not a substitute.

Limits the boot binding must enforce
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PROVISIONAL, pending coordinator approval. The complete canonical success-mark envelope is at most 4,096 bytes inclusive; the core and mark payloads are each at most 3,072 bytes; boot objects have depth at most 8, at most 32 properties, arrays at most 32 entries, and at most 32 checks. The complete core envelope maximum is not stated by the candidate and is BLK-13; the U-Boot container reserves 4,096 bytes for it and rejects a ratified bound above that as a design conflict requiring a new ruling rather than silently widening. ``Digest`` is ``sha256:`` plus 64 lowercase hexadecimal characters. ``SlotId`` is exactly ``slot-a``, ``slot-b``, or ``recovery``. ``BoardId`` is ``apple:`` plus a lowercase token of 1 to 64 bytes. ``Generation`` and counters are unsigned 64-bit values that never wrap or reset; the value 18,446,744,073,709,551,615 is a hard stop that returns ``BOOT_COUNTER_FAILURE`` before any increment.

Failure vocabulary used by the boot consumer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PROVISIONAL, pending coordinator approval. The boot HOLD/HALT set includes ``BOOT_MARKER_AUTH_FAILURE``, ``BOOT_CONTEXT_MISMATCH``, ``BOOT_COUNTER_FAILURE``, ``BOOT_REQUIRED_CHECK_FAILURE``, ``BOOT_FALLBACK_FAILURE``, ``TRUST_BOUNDARY_FAILURE``, ``BOOT_DEVICE_BINDING_FAILURE``, ``BOOT_RECORD_DIGEST_FAILURE``, ``BOOT_RECORD_SOURCE_FAILURE``, ``BOOT_REPLAY_RESERVATION_FAILURE``, ``BOOT_PROJECTION_FAILURE``, ``BOOT_PROFILE_FAILURE``, ``BOOT_FRESHNESS_FAILURE``, ``BOOT_PROMOTION_FAILURE``, ``BOOT_STORAGE_IDENTITY_FAILURE``, ``BOOT_CANDIDATE_CONFLICT``, ``BOOT_STATE_CONSISTENCY_FAILURE``, ``BOOT_RECOVERY_IDENTITY_FAILURE``, and ``BOOT_RECORD_COMMIT_FAILURE``. A HOLD/HALT code never produces success, never selects an unverified slot, and never uses a fallback that bypasses its predecessor authority. Every other code encountered by U-Boot (``PARSE_SCHEMA_FAILURE``, ``UNKNOWN_FIELD``, ``DUPLICATE_SEMANTIC_KEY``, ``CANONICALIZATION_FAILURE``, ``SIGNATURE_CONTEXT_MISMATCH``, ``TRUST_FAILURE``, ``EXPIRY_OR_REPLAY_FAILURE``, ``MANIFEST_EXPIRY_FAILURE``, ``CROSS_DOCUMENT_MISMATCH``, ``DOCUMENT_ID_REUSE``, ``DOCUMENT_ID_FORK``, ``BINDING_INTEGRITY_FAILURE``, ``RESOURCE_LIMIT``) is a reject: the object is treated as absent. U-Boot maps these to the 16-bit local codes in `Local failure codes`_ for storage in the durable record; the mapping is one-to-one and the string code is what U-Boot prints.

Authority separation
--------------------

The authority model is deliberately bounded. F-07 owns promotion, the ``boot-runtime`` signer owns success evidence, and U-Boot owns one combined selection-and-transition state machine. The combined U-Boot authority is not split into independent powers and makes no independent-powers claim: it may select or write only from authenticated inputs, only through the total transitions below, and only after the F-02/F-03, storage, manifest, profile, and F-07 gates pass. Success signing and F-07 promotion remain distinct authorities.

.. list-table::
   :header-rows: 1
   :widths: 24 24 52

   * - Authority
     - Sole holder
     - What the holder may and may not do
   * - Release-channel authority
     - F-07 promotion terminal
     - Alone copies an already-built candidate digest into the ``stable`` namespace. U-Boot never contacts a channel and never decides what is stable; it checks only that the immutable manifest staged in a slot carries ``channel`` equal to its compiled profile and verifies under ``manifest-release``.
   * - Boot selection and transition authority
     - U-Boot ``omarchy release`` state machine
     - One explicitly bounded authority: it selects and writes only the exact state-machine transition whose authenticated inputs it has validated. It never edits a manifest, chooses a component version, scans for alternatives, accepts a signer, or invents a recovery identity. The OS never writes PENDING, ACCEPTED, or FAILED. The only non-U-Boot write is the pre-boot provisioning of an unkeyed record that cannot express acceptance (T-01).
   * - Boot success authority
     - ``boot-runtime`` signer inside the booted OS, evaluated by U-Boot
     - Emits a signed ``boot-health/v1`` core and a separately signed ``boot-success-mark/v1`` into the success-mark container. It cannot mutate boot-control copies through that container, and the container has no field that means accepted. U-Boot validates ``Trusted<BootHealthCore>`` and then the separate success mark on the next boot and alone performs the PENDING to ACCEPTED transition.

Missing, stale, replayed, forged, mismatched, or oversize marks fail closed: the pending slot keeps consuming its bounded attempt budget, the last-known-good slot is never consumed by a mark failure, and no mark failure can change the last-known-good descriptor.

Durable boot-control record
---------------------------

Storage identity
~~~~~~~~~~~~~~~~

Two Omarchy-owned GPT partitions on one exact physical NVMe disk as the Apple ESP hold one copy each of the boot-control record (BCR). The BSM and every slot artifact must resolve through that same authenticated physical disk identity. Both BCR partitions carry the partition type GUID BLK-01, the partition names ``OMARCHY-BCR-0`` and ``OMARCHY-BCR-1``, and a size of exactly 1,048,576 bytes. Only bytes 0 through 4,095 of each partition are ever read or written; the remainder is zero at provisioning and is never read. The two partitions are provisioned by the installer under an approved plan whose scope names their stable IDs (``partition:v1:sha256:...`` under the F-02 typed storage identity) and are never created, moved, resized, or deleted by U-Boot.

U-Boot first gathers all candidate disks and validates GPT headers. It then resolves the ESP by gathering every partition whose UUID equals the predecessor-supplied UUID, requiring exactly one match, and comparing the live GPT disk GUID, partition GUID, typed parent, and stable ID to the authenticated installation identity. It resolves both BCR partitions and the BSM in the same way and requires the live GPT disk GUID and typed parent/stable-ID relation to be equal across ESP, BCR-0, BCR-1, BSM, the selected slot directory, and the trusted record. Zero matches, multiple matches, a cross-disk tuple, a parent mismatch, a GUID mismatch, or a stable-ID mismatch is ``BOOT_STORAGE_IDENTITY_FAILURE``; the failure occurs before reading any slot manifest or artifact. Resolution is repeated immediately before every write. A raw device path, environment value, glob, enumeration order, or first-match fallback is never accepted as the write target.

Freshness and installation binding
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The on-disk sequence is not an anti-rollback authority. The future-ratified F-02/F-03 ``DeviceInstallationBinding`` supplies the authenticated current-device and installation identity and an independently monotonic per-device/installation floor. Its exact preimage is ``ASCII("omarchy-device-installation-binding/v1") || 0x00 || JCS(I)`` where ``I`` is the closed object containing the board ID, installation ID, physical GPT disk GUID, ESP/BCR/BSM stable IDs and partition GUIDs, source authority ID, schema-set digest, and binding generation. The independent floor authority supplies a signed ``MonotonicFloorReceipt`` whose exact preimage is ``ASCII("omarchy-monotonic-floor/v1") || 0x00 || JCS(F)`` where ``F`` contains the installation identity digest, floor value, reservation ID, authority ID, binding digest, issued-at, expiry, and replay state. These preimages and fields are provisional until F-02/F-03 ratification; no local alias or guessed value is accepted.

The exact ordering is: (1) verify the ratified F-02/F-03 schemas and ``Trusted<TrustContext>``; (2) authenticate the current-device/installation binding against live GPT identity; (3) obtain the independent floor and reject a missing, expired, revoked, or regressed receipt; (4) read both BCR copies and construct ``Trusted<AtomicBootRecord>`` only when its ``last_counter`` is at least the external floor and its source record and binding digest match; (5) reserve the next counter in the independent authority, burning the reservation on every later failure; (6) write and read back both committed copies carrying that exact ``last_counter`` and ``replay_reservation``; (7) commit the reservation in the independent authority; and only (8) install BootContext or launch. A failed reserve, write, read-back, or floor commit is ``BOOT_COUNTER_FAILURE`` or ``BOOT_DEVICE_BINDING_FAILURE``, no write or launch is permitted after the failure, and the terminal result is ``HOLD/HALT``. A stale full-disk image or clone cannot lower the external floor or change the current-device identity.

.. _atomic-boot-record-mapping:

Canonical F-02 ``AtomicBootRecord`` mapping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The BCR is the physical storage encoding of the future-ratified F-02 ``AtomicBootRecord``; it is not a second record type and no consumer-local alias is permitted. The logical record ``R`` is authenticated and canonicalized before storage. Every field below is mandatory in the generated F-02 binding, and a missing or mismatched field prevents ``Trusted<AtomicBootRecord>`` construction.

.. list-table::
   :header-rows: 1
   :widths: 24 28 20 28

   * - F-02 member
     - Exact canonical preimage or source
     - BCR storage projection
     - Constructor guard and failure
   * - ``journal_schema``
     - ``R.journal_schema`` in the closed F-02 record; exact ratified schema identifier
     - BCR-22, canonical bytes
     - Exact equality before record verification; ``BINDING_INTEGRITY_FAILURE`` at ``bcr.atomic_record.journal_schema``
   * - ``commit_state``
     - ``R.commit_state`` from the F-02 journal, never inferred from copy order
     - BCR-23, canonical enum bytes
     - Only the ratified committed value constructs the trusted record; otherwise ``BOOT_RECORD_COMMIT_FAILURE``
   * - ``record_digest``
     - ``sha256(ASCII("omarchy-boot-lineage-record/v1") || 0x00 || JCS(R without record_digest))``
     - BCR-24, raw SHA-256 bytes
     - Recompute before any state selection; mismatch is ``BOOT_RECORD_DIGEST_FAILURE``
   * - ``authenticated_source_record``
     - The exact F-02 authenticated journal source record and its signer/context, not a local BCR reconstruction
     - BCR-25 source-record digest and BCR-26 source authority binding
     - Verify source record, signer, context, freshness, and installation identity; failure is ``BOOT_RECORD_SOURCE_FAILURE``
   * - ``last_counter``
     - Monotonic value in ``R`` and the independently authenticated floor receipt
     - BCR-27, u64; never the local sequence alias
     - Require ``R.last_counter >= external_floor`` and reserve before transition; failure is ``BOOT_COUNTER_FAILURE``
   * - ``replay_reservation``
     - Exact F-02 reservation object, including reservation ID, counter, authority, and consumed state
     - BCR-28 reservation digest plus the authenticated source record
     - Verify reservation is fresh, unconsumed, and equal to ``last_counter``; failure is ``BOOT_REPLAY_RESERVATION_FAILURE``
   * - ``permanent_tombstones``
     - Append-only F-02 document-lineage history covering every prior document ID, payload digest, generation, and lineage
     - BCR-29 tombstone-root digest; history is durable and authenticated, never descriptor-local
     - Enforce one ``document_id`` to one ``payload_digest`` and permanent non-reuse; failure is ``DOCUMENT_ID_REUSE`` or ``DOCUMENT_ID_FORK``
   * - ``Trusted<AtomicBootRecord>`` constructor
     - Generated F-02 constructor over the verified canonical journal record and ``Trusted<TrustContext>``
     - No local constructor or consumer-local alias
     - The only result is ``Trusted<AtomicBootRecord>`` or no value; unavailable/mismatched F-02/F-03 authority is ``HOLD/HALT``

The current fixed-offset fields below are a storage proposal pending that ratification. Until the mapping is ratified, BCR-07 ``sequence`` is evidence only, cannot satisfy ``last_counter``, and no release transition is admitted. Permanent tombstones are retained across every slot-generation rewrite and across both copies; descriptor recycling never deletes lineage history.

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
     - 1 through 8, copied from the complete manifest-declared profile's ``retry_limit``; the full profile is separately bound by ``profile_digest`` and ``profile_id`` before ``Trusted<BootHealthCore>``; a profile value of 0 or above 8 is ``BOOT_PROFILE_FAILURE`` and the slot cannot become PENDING
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
     - Sole derived payload digest ``sha256(JCS(payload))`` of the slot manifest; it is not a wire/envelope property and is zero only when EMPTY
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

Canonical AtomicBootRecord projection, bytes 1,024 through 1,227:

.. list-table::
   :header-rows: 1
   :widths: 9 8 6 9 26 42

   * - ID
     - Offset
     - Size
     - Type
     - Field
     - Bound
   * - BCR-22
     - 1024
     - 32
     - ascii
     - ``journal_schema``
     - Exact canonical F-02 ``R.journal_schema``; NUL padded and zero terminated
   * - BCR-23
     - 1056
     - 4
     - u32
     - ``commit_state``
     - Exact canonical F-02 committed-state value; no inference from copy order
   * - BCR-24
     - 1060
     - 32
     - digest
     - ``record_digest``
     - Recomputed from the exact record-digest preimage in `atomic-boot-record-mapping`_
   * - BCR-25
     - 1092
     - 32
     - digest
     - ``authenticated_source_record_digest``
     - Digest of the exact authenticated F-02 source record bytes
   * - BCR-26
     - 1124
     - 32
     - digest
     - ``source_authority_binding_digest``
     - Digest of the source authority and signer/context binding
   * - BCR-27
     - 1156
     - 8
     - u64
     - ``last_counter``
     - Exact external-floor-bound F-02 counter; not BCR-07 and never reset or reused
   * - BCR-28
     - 1164
     - 32
     - digest
     - ``replay_reservation_digest``
     - Digest of the exact F-02 replay reservation object bound to BCR-27
   * - BCR-29
     - 1196
     - 32
     - digest
     - ``permanent_tombstone_root``
     - Authenticated append-only document-lineage root; zero is valid only for a never-staged provisioned record
   * - BCR-30
     - 1228
     - 32
     - digest
     - ``pending_profile_digest``
     - Exact digest of the complete manifest-declared BootCheck profile for the PENDING descriptor; zero otherwise
   * - BCR-31
     - 1260
     - 32
     - digest
     - ``pending_profile_id_digest``
     - Digest of the exact profile ID bound to BCR-30; zero otherwise
   * - BCR-32
     - 1292
     - 32
     - digest
     - ``pending_profile_source_digest``
     - Digest of the authenticated profile source evidence/record; zero otherwise

Bytes 1,324 through 4,047 are reserved and zero. Trailer region, bytes 4,048 to 4,095:

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
   * - 20
     - ``BOOT_DEVICE_BINDING_FAILURE``
     - Current physical device or installation identity does not equal the authenticated binding
   * - 21
     - ``BOOT_RECORD_DIGEST_FAILURE``
     - F-02 ``record_digest`` or its exact canonical preimage does not verify
   * - 22
     - ``BOOT_RECORD_SOURCE_FAILURE``
     - Authenticated source record, authority, signer, context, or source freshness does not verify
   * - 23
     - ``BOOT_REPLAY_RESERVATION_FAILURE``
     - F-02 replay reservation is absent, stale, consumed, duplicated, or differs from ``last_counter``
   * - 24
     - ``BOOT_PROJECTION_FAILURE``
     - Generated F-02 manifest projection differs in length, key, order, type, or any field
   * - 25
     - ``BOOT_PROFILE_FAILURE``
     - Manifest-declared BootCheck profile differs from the core or compiled profile
   * - 26
     - ``BOOT_FRESHNESS_FAILURE``
     - Ratified clock, floor, issued-at, expiry, or not-before rule cannot be proven
   * - 27
     - ``BOOT_PROMOTION_FAILURE``
     - F-07 promotion receipt is absent, stale, unauthorized, incomplete, replayed, or mismatched
   * - 28
     - ``BOOT_STORAGE_IDENTITY_FAILURE``
     - Zero or multiple ESP matches, duplicate UUID, cross-disk tuple, parent mismatch, or stable-ID mismatch
   * - 29
     - ``BOOT_CANDIDATE_CONFLICT``
     - More than one F-07-authorized candidate is eligible or candidates tie/conflict
   * - 30
     - ``DOCUMENT_ID_REUSE``
     - A document ID is presented with a second payload digest after durable staging
   * - 31
     - ``DOCUMENT_ID_FORK``
     - A document ID has divergent payload digests in the durable lineage history
   * - 32
     - ``RELEASE_COMMAND_CLOSURE_FAILURE``
     - The generated command allowlist finds an enabled command outside the declared diagnostic set
   * - 33
     - ``BOOT_PROVENANCE_FAILURE``
     - A typed source, artifact, recipe, toolchain, report, or promotion relation is absent or mismatched
   * - 34
     - ``BOOT_STATE_CONSISTENCY_FAILURE``
     - The selected slot or descriptor Cartesian state is unknown, contradictory, or not a legal state
   * - 35
     - ``BOOT_RECOVERY_IDENTITY_FAILURE``
     - Immutable compiled or independently authenticated recovery identity/digest is unavailable or mismatched
   * - 36
     - ``BOOT_RECORD_COMMIT_FAILURE``
     - A BCR/BSM durable write, flush, or read-back did not complete exactly
   * - 37
     - ``CI_CONFORMANCE_FAILURE``
     - A required generated fixture, validator, or CI gate is absent, fails, or is unobserved
   * - 38
     - ``QUALIFICATION_FAILURE``
     - Required board/profile/physical qualification evidence is absent or fails
   * - 39
     - ``RELEASE_PROFILE_FAILURE``
     - The resolved release configuration is absent, mismatched, or not fully closed

The codes 14 through 39 are U-Boot-local diagnostic codes for conditions the F-02 vocabulary does not name at this boundary. They are never written into an F-02 payload; the OS-side writer maps a U-Boot local code it observes in the BootContext to ``TRUST_BOUNDARY_FAILURE`` if it must report one. Each code has one first-failure path, phase, result, and terminal action in `Failure code/path/phase/result matrix`_.

Failure code/path/phase/result matrix
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following is the stable mapping used by every transition, copy case, success-mark case, hostile fixture, residual, and blocked constant. The path is the first canonical input path or physical resolver; later checks cannot replace its result. ``REJECT`` means the object is absent, ``HOLD`` means no success or new selection, and ``HALT`` means no write or launch.

.. list-table::
   :header-rows: 1
   :widths: 24 30 20 26

   * - Code
     - First-failure path
     - Phase
     - Result
   * - ``BINDING_INTEGRITY_FAILURE``
     - ``contract.f02_f03.schema_set_digest``
     - ratification
     - ``REJECT/HOLD/HALT``; no constructor, write, or launch
   * - ``BOOT_DEVICE_BINDING_FAILURE``
     - ``storage.device_installation_binding``
     - device binding
     - ``HOLD/HALT``; no slot read or write
   * - ``BOOT_COUNTER_FAILURE``
     - ``atomic_record.last_counter`` or ``monotonic_floor``
     - freshness/reservation
     - ``HOLD/HALT``; reservation is burned and no launch
   * - ``BOOT_RECORD_DIGEST_FAILURE``
     - ``bcr.atomic_record.record_digest``
     - record authentication
     - ``REJECT/HALT``; no copy selected
   * - ``BOOT_RECORD_SOURCE_FAILURE``
     - ``bcr.atomic_record.authenticated_source_record``
     - record authentication
     - ``REJECT/HALT``; no copy selected
   * - ``BOOT_REPLAY_RESERVATION_FAILURE``
     - ``bcr.atomic_record.replay_reservation``
     - replay admission
     - ``HOLD/HALT``; no transition
   * - ``BOOT_PROJECTION_FAILURE``
     - ``manifest.payload.<projection>``
     - manifest projection
     - ``REJECT/HALT``; no trusted manifest
   * - ``BOOT_PROFILE_FAILURE``
     - ``boot_health.profile.<field>``
     - core admission
     - ``REJECT/HOLD``; no trusted core
   * - ``BOOT_REQUIRED_CHECK_FAILURE``
     - ``boot_health.checks[<index>]``
     - core admission
     - ``HOLD``; no success mark accepted
   * - ``BOOT_FRESHNESS_FAILURE``
     - ``boot_health.freshness`` or ``success_mark.freshness``
     - freshness
     - ``HOLD/HALT``; no success
   * - ``BOOT_PROMOTION_FAILURE``
     - ``manifest.payload.promotion_receipt``
     - promotion admission
     - ``REJECT/HALT``; no candidate staging
   * - ``BOOT_STORAGE_IDENTITY_FAILURE``
     - ``storage.esp.matches`` or ``storage.parent_relation``
     - physical resolution
     - ``HOLD/HALT``; no slot files read
   * - ``BOOT_CANDIDATE_CONFLICT``
     - ``candidate_set``
     - stage detection
     - ``HOLD/HALT``; no PENDING write
   * - ``DOCUMENT_ID_REUSE`` or ``DOCUMENT_ID_FORK``
     - ``atomic_record.permanent_tombstones``
     - lineage admission
     - ``REJECT/HALT``; no staging
   * - ``RELEASE_COMMAND_CLOSURE_FAILURE``
     - ``release_command_allowlist.enabled_command``
     - build/design gate
     - build fails; no release admission
   * - ``BOOT_PROVENANCE_FAILURE``
     - ``provenance.<relation>``
     - provenance admission
     - ``REJECT/HALT``; no trusted artifact
   * - ``BOOT_STATE_CONSISTENCY_FAILURE``
     - ``atomic_record.selected_slot`` and descriptors
     - state validation
     - ``HOLD/HALT``; no write or launch
   * - ``BOOT_RECOVERY_IDENTITY_FAILURE``
     - ``recovery.immutable_identity``
     - recovery admission
     - ``HALT``; no emergency fallback
   * - ``BOOT_RECORD_COMMIT_FAILURE``
     - ``bcr.copy[0|1].readback`` or ``storage.flush``
     - durable commit
     - ``HOLD/HALT``; no launch
   * - ``CI_CONFORMANCE_FAILURE``
     - ``ci.required_fixture``
     - CI gate
     - ``REJECT/HALT``; no promotion
   * - ``QUALIFICATION_FAILURE``
     - ``qualification.board_profile``
     - physical gate
     - ``REJECT/HALT``; no promotion
   * - ``RELEASE_PROFILE_FAILURE``
     - ``release.config``
     - release-profile gate
     - ``REJECT/HALT``; no release build

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
     - Writes both copies with the exact ratified F-02 ``journal_schema``, ``commit_state``, authenticated source record, device/installation binding, floor receipt, replay reservation, and permanent-tombstone root; ``sequence = 1`` is evidence only, ``auth_algorithm = 1`` is provisioning-only, ``slot-a`` PENDING with ``attempts_used = 0``, ``attempt_counter = 0``, ``slot_generation = 1``, the staged manifest and complete profile digests, ``slot-b`` EMPTY, ``recovery`` PINNED with the immutable recovery identity, ``last_known_good_slot = 0``, and ``selected_slot = 1``. Any other content in an unkeyed record is invalid. If the canonical F-02/F-03 source, device binding, or floor cannot be authenticated, provisioning is not a release admission and the result is HOLD/HALT; there is no recovery-OS rewrite path.
   * - T-02
     - U-Boot
     - unkeyed to keyed
     - Folded into the first U-Boot commit only after the ratification, device-binding, floor, and source-record gates: the record is rewritten with ``auth_algorithm = 2``, exact F-02 ``commit_state = committed``, a fresh MAC, ``record_digest``, and the reserved counter. In a non-release profile the record stays at algorithm 1 and cannot launch or produce a trusted BootContext.
   * - T-03
     - U-Boot
     - EMPTY, FAILED, or non-last-known-good ACCEPTED to PENDING (stage detection)
     - Exactly one eligible candidate exists. Its manifest passes the generated F-02 projection validator, the exact F-07 promotion receipt, board/channel/artifact/policy/anti-downgrade checks, complete BootCheck profile check, and document-lineage tombstone check before ``Trusted<PlatformManifest>`` is constructed. It names the current last-known-good manifest ID in ``components.boot_stack.rollback.previous_manifest_ids`` (or ``last_known_good_slot = 0``), every boot-path artifact verifies, and ``retry_limit`` is 1 through 8. If zero candidates exist, no staging occurs; if more than one exists, all are rejected with ``BOOT_CANDIDATE_CONFLICT`` and no PENDING write. Effect for exactly one: ``slot_generation + 1``, new ``lineage_id``, ``attempt_counter = 0``, ``attempts_used = 0``, ``attempt_limit``, complete profile digests, manifest/artifact digests, ``rollback_set_digest``, and ``selected_slot`` set to this slot. The prior document identity is appended to the permanent tombstone history (T-12), never discarded by descriptor overwrite.
   * - T-04
     - U-Boot
     - PENDING to PENDING (attempt consumption)
     - ``selected_slot`` is exactly the PENDING descriptor, ``attempts_used < attempt_limit``, and the independent authority durably reserves the next ``last_counter`` before any write. Effect: ``attempts_used + 1``, ``attempt_counter = reservation.counter``, ``last_counter = reservation.counter``, ``replay_reservation``, ``pending_time_unix``. Both copies are committed and read back before the reservation is committed and before BootContext installation or launch.
   * - T-05
     - U-Boot
     - PENDING to ACCEPTED
     - ``Trusted<BootHealthCore>`` was constructed only after the complete manifest-declared profile, source evidence, and freshness checks, and the separate success-mark validator accepts the exact ``attempt_counter``, core digest, descriptor, F-07 receipt, and rollback set. Effect: state ACCEPTED, ``last_known_good_slot`` set to this slot, BCR-S12, BCR-S13, BCR-S14 recorded, ``attempts_used`` retained as evidence, and the permanent lineage history extended. The former last-known-good slot remains ACCEPTED but is no longer last-known-good and becomes eligible for T-03. Committed before the mark container is cleared.
   * - T-06
     - U-Boot
     - PENDING to FAILED (budget exhausted)
     - ``attempts_used = attempt_limit`` and no valid success mark. Effect: state FAILED, ``slot_failure_code = 1``, ``selected_slot`` set to the exact current LKG when non-zero, otherwise PINNED recovery. The FAILED descriptor keeps its identity and the permanent tombstone keeps its document ID and payload digest so the same candidate cannot be retried or forked.
   * - T-07
     - U-Boot
     - PENDING to FAILED (explicit OS failure)
     - A ``Trusted<BootHealthCore>`` for exactly ``attempt_counter`` passed the complete profile validation and carries ``success = false`` and ``fallback.decision = recover`` with ``target_slot`` in the recomputed rollback set. Effect as T-06 with ``slot_failure_code`` from the core ``fallback.failure_code`` mapping. A core with ``decision = hold`` leaves the slot PENDING and the next boot performs T-04; an invalid or stale core is not an explicit failure and cannot alter state.
   * - T-08
     - U-Boot
     - PENDING to FAILED (pre-launch validation failure)
     - The pending slot's manifest is missing, unsigned, expired, fails projection or F-07 promotion validation, not the recorded derived payload digest, or any boot-path artifact fails size or digest. Effect as T-06 with the first stable code from the failure matrix; a missing canonical F-02/F-03 dependency is HOLD/HALT and never a fallback.
   * - T-09
     - U-Boot
     - record-level only
     - The exact current LKG slot's on-disk manifest derived payload digest no longer equals its ACCEPTED descriptor, its F-07 receipt, or a boot-path artifact fails. Effect: ``record_failure_code = 19``, select only the independently authenticated PINNED recovery identity; the descriptor and lineage history are not erased. If the commit fails, recovery is permitted only under the immutable recovery identity and full checks; otherwise HALT without a write.
   * - T-10
     - U-Boot
     - none
     - Manual diagnostic boot (``omarchy try`` or ``omarchy recovery``) has no release authority and writes neither copy. Automatic recovery selection is a release state-machine outcome, not T-10, and still requires the immutable recovery identity and full checks.
   * - T-11
     - U-Boot
     - none (read-repair)
     - A stale or invalid copy is rewritten from the selected valid record with its own ``copy_index``, tag, and CRC. ``sequence`` is unchanged. Performed after selection and before any transition commit; a repair failure is logged and does not stop a boot that requires no transition.
   * - T-12
     - U-Boot
     - descriptor overwrite (tombstone)
     - Part of T-03 and T-05: append the prior document ID, one and only one payload digest, generation, lineage, and manifest digest to the authenticated F-02 permanent-tombstone history before replacing a descriptor. An older mark or a document-ID fork can never match again; recycling a descriptor does not delete the history.

State validation is total and runs before selection or any write. ``selected_slot`` may be only a PENDING slot, the exact current LKG ACCEPTED descriptor, or the PINNED recovery descriptor. ``selected_slot`` may never name FAILED, EMPTY, an ACCEPTED descriptor that is not the LKG, an unknown value, or a contradictory descriptor. A PENDING or FAILED descriptor has ``attempt_limit`` in range and ``attempts_used`` within it; ``last_known_good_slot`` is zero or names exactly one ACCEPTED slot; at most one of ``slot-a`` and ``slot-b`` is PENDING; the recovery descriptor is EMPTY or PINNED; an ACCEPTED descriptor has non-zero BCR-S12, BCR-S13, and BCR-S14; and an unkeyed record has exactly the T-01 shape. Every invalid Cartesian combination returns ``BOOT_STATE_CONSISTENCY_FAILURE``, records the first canonical state path, performs no write or launch, and terminates in ``HOLD/HALT``. No assertion that a writer "derived" a field substitutes for validation.

State validation Cartesian coverage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The design gate generates the complete Cartesian set ``selected_slot ∈ {0, slot-a, slot-b, recovery, unknown}`` × ``selected_descriptor_state ∈ {EMPTY, PENDING, ACCEPTED, FAILED, PINNED}`` × ``last_known_good_slot ∈ {0, slot-a, slot-b, recovery-or-unknown}`` × ``other_slot_state ∈ {EMPTY, PENDING, ACCEPTED, FAILED, PINNED}`` × ``recovery_state ∈ {EMPTY, PINNED, PENDING, ACCEPTED, FAILED}``: 2,500 cases. The matrix below supplies the total predicate for every selected-slot/state pair; the generated cross-product applies the LKG, other-slot, recovery, counter, and descriptor invariants to every row.

.. list-table::
   :header-rows: 1
   :widths: 18 16 16 16 16 18

   * - ``selected_slot``
     - EMPTY
     - PENDING
     - ACCEPTED
     - FAILED
     - PINNED
   * - 0
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
   * - ``slot-a``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 0; action=T-04 then launch; terminal=``LAUNCH``
     - code 0; action=LKG verify; terminal=``LAUNCH``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
   * - ``slot-b``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 0; action=T-04 then launch; terminal=``LAUNCH``
     - code 0; action=LKG verify; terminal=``LAUNCH``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
   * - ``recovery``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 0; action=immutable-recovery verify then launch; terminal=``LAUNCH``
   * - unknown
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``
     - code 34; action=none; terminal=``HOLD/HALT``

The matrix is exhaustive, not illustrative: each generated case records ``(selected_slot, descriptor states, last_known_good_slot, code, action, terminal_state)`` and the acceptance gate fails if the count is not exactly 2,500, if any tuple is missing or duplicated, or if any legal row lacks a deterministic launch/transition result.

Two-copy selection and recovery
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On every boot U-Boot first passes the ratification and device-binding gates, gathers all physical storage matches, and verifies the single same-disk ESP/BCR/BSM tuple before reading slot files. It then reads both copies. A copy is valid only if every check in `Record encoding`_, `atomic-boot-record-mapping`_, `Local failure codes`_, and the total state rules passes, including the live GPT identity and external floor. Selection precedence is: exact F-02 committed record with a valid independent floor and source record over a provisioning-only record; higher authenticated ``last_counter`` over lower; and among equal counters, bytewise equality outside the three copy-specific fields. BCR-07 ``sequence`` and timestamps never participate in authority selection.

.. list-table::
   :header-rows: 1
   :widths: 8 34 58

   * - ID
     - Observation
     - Decision
   * - C2-01
     - Both valid, equal authenticated ``last_counter``, equal content
     - Use the record; no repair
   * - C2-02
     - Both valid, equal authenticated ``last_counter``, different content
     - ``OMARCHY_BCR_DIVERGENT``; no write of either copy and no record is selected. Recovery is permitted only from the one immutable compiled or independently authenticated recovery identity, not from either copy; if it is unavailable or any full check fails, ``BOOT_RECOVERY_IDENTITY_FAILURE`` and HALT
   * - C2-03
     - Both valid, authenticated ``last_counter`` values differ by exactly 1
     - Use the higher; T-11 repairs the lower (interrupted commit between copies)
   * - C2-04
     - Both valid, authenticated ``last_counter`` values differ by more than 1
     - Use the higher; T-11 repairs the lower; diagnostic note that one copy lagged more than one commit
   * - C2-05
     - One valid, the other fails any validity check
     - Use the valid copy; T-11 repairs the other
   * - C2-06
     - One valid, the other partition cannot be resolved by the identity tuple
     - Use the valid copy read-only only when its external floor, source record, and device binding verify; no repair. Commits are impossible because a commit needs both copies, so a transition is refused with ``BOOT_STORAGE_IDENTITY_FAILURE``; the exact current LKG may boot only if all launch checks pass, otherwise the immutable recovery identity is required or HALT
   * - C2-07
     - Zero valid, both partitions resolved
     - No write and no BCR is selected. Verify one immutable compiled or independently authenticated recovery identity/digest plus the complete manifest, artifact, board, policy, anti-downgrade, F-02/F-03, and F-07 rules. If the identity or any dependency is unavailable or mismatched, ``BOOT_RECOVERY_IDENTITY_FAILURE`` and HALT. There is no board-plus-signature fallback and no hidden emergency envelope
   * - C2-08
     - GPT invalid or neither partition resolved
     - No write; ``BOOT_STORAGE_IDENTITY_FAILURE`` and HALT with the GPT or exact-match failure printed; the ESP and all slot artifacts are untrusted
   * - C2-09
     - A copy whose ``copy_index`` or partition GUID does not match the partition it was read from
     - That copy is invalid (transplant); the case reduces to C2-05 or C2-07
   * - C2-10
     - A keyed valid copy and an unkeyed valid copy
     - The keyed, floor-bound, source-authenticated copy wins only when all identity checks pass; the unkeyed copy is treated as provisioning-only and cannot authorize a release transition. Re-provisioning requires an approved installer plan and fresh external binding; destroying keyed copies is never a release recovery operation
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

1. Re-resolve both partitions with the full identity tuple; abort with ``BOOT_RECORD_COMMIT_FAILURE`` and no write if either fails.
2. Reserve the next independent monotonic ``last_counter`` and replay reservation; a reservation is burned on every failure and is never reused.
3. Build the complete canonical F-02 record in memory with the exact ``journal_schema``, ``commit_state``, source record, device binding, ``record_digest``, ``last_counter``, replay reservation, profile projection, and permanent tombstone root; compute the trailer sequence, tag, and CRC for copy 0.
4. Write 4,096 bytes at block 0 of copy 0; issue a device flush (NI-08); read the block back and compare bytewise; abort with ``BOOT_RECORD_COMMIT_FAILURE`` on any mismatch, leaving no new authority.
5. Recompute tag and CRC for copy 1; write, flush, read back, and compare. A failure leaves the new state uncommitted and the current boot does not launch; the next boot may use only the previously committed F-02 record or immutable recovery identity according to C2.
6. Commit the independent floor reservation and verify its authenticated receipt. A failure is ``BOOT_COUNTER_FAILURE`` with no launch.
7. Only after both copies and the independent reservation are durable does U-Boot install BootContext and launch.

Writer serialization: U-Boot runs single-threaded, and the writer keeps a static in-progress flag; a re-entrant call returns code 17 without writing. Only ``omarchy release`` reaches the writer. The diagnostic subcommands, the EFI runtime, and any other command have no path to it. Alternate-copy ordering is fixed (copy 0 then copy 1) so that every interrupted state is one of C2-11 through C2-13.

Power-loss points that the sandbox suite must inject, each with the expected next-boot case: before step 1 (previous state), while building the record before step 4 (previous state), during step 4 copy 0 (C2-11), between the copy 0 read-back and step 5 (C2-03), during step 5 copy 1 (C2-12), after step 7 before the payload runs (attempt consumed, C2-01), during GRUB (attempt consumed, no mark), during Linux before the mark write (no mark), during the mark container write (SM-04), after the mark write before the next boot (SM-16 on the next boot), during the T-05 commit (C2-11 or C2-12 with the mark still present, SM-17 on the following boot), and during the mark clear (SM-17). Every cut point captures both copies, the external reservation state, the BSM header, and the terminal code; no cut point is evidence until the fixture executes.

Success-mark transport
----------------------

The success mark and the boot-health core travel in a third Omarchy-owned GPT partition, the boot success-mark container (BSM). The OS writes it; U-Boot reads, validates, and clears it. It has no state field that means accepted, so writing it cannot accept a slot.

Container identity and encoding
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The partition carries type GUID BLK-02, the name ``OMARCHY-BSM``, and a size of exactly 1,048,576 bytes; its unique GUID is recorded in BCR-12 and is part of the BCR authenticated region. Before reading any region, U-Boot compares the live BSM disk GUID and typed parent/stable ID to BCR-09/BCR-12 and to the one resolved ESP and both BCR partitions. Only bytes 0 through 12,287 are used: a 4,096-byte header block, a 4,096-byte core region, and a 4,096-byte mark region. All integers are little-endian.

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
     - 136
     - 3956
     - u8[3956]
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
   * - BSM-19
     - 88
     - 16
     - guid
     - ``disk_guid``
     - Must equal the live GPT disk GUID and BCR-09; a mismatch is a cross-disk tuple
   * - BSM-20
     - 104
     - 32
     - digest
     - ``parent_stable_id_digest``
     - Digest of the typed F-02 parent/stable-ID relation for this BSM; must equal the authenticated installation binding

Canonical bytes are the exact JCS envelope bytes whose signature was produced by the ``boot-runtime`` key; the container adds no framing inside the regions. Authentication is entirely the Ed25519 envelope signature verified against the ``boot-runtime`` role binding in the F-03 trust context that U-Boot carries (BLK-05); the container's CRCs are for torn-write detection only. The core and mark remain separate authenticated documents, and neither container metadata nor a derived payload digest grants acceptance.

Producer procedure (OS side, owned by the Linux consumer under K-01 and the health writer under P-05): construct ``Trusted<AtomicBootRecord>`` and then ``Trusted<BootContext>`` from the firmware handoff as described in `BootContext transport`_; bind the complete manifest-declared profile, including allowed classes, exact check set/order, measurements, bounds, limits, source record, status, profile digest, and freshness; run the checks; sign the core; validate the core as the input to a separate mark operation; sign the mark with ``attempt_counter`` equal to the BootContext counter, ``marker_generation`` greater than the previous accepted generation for the lineage, and a fresh ``marker_replay_id``; write the core region with a single aligned 4,096-byte write and flush; write the mark region the same way and flush; write the header block with ``container_state = 1`` and both region CRCs and flush; read back all three blocks and compare. The header is written last so it never references bytes that were not durably written. The writer refuses to run when the BootContext, canonical F-02/F-03 authority, or complete profile is absent or invalid, so a diagnostic boot produces no container write.

Consumer procedure (U-Boot, inside ``omarchy release`` and only after two-copy selection): read the three blocks; validate the header, physical relation, and region CRCs; verify and construct the core first under the boot binding; bind its complete profile and freshness; then independently verify and construct the mark and compare it to the trusted core, pending descriptor, F-07 receipt, reservation, and rollback set. Only then commit T-05 or T-07, or leave the state unchanged; clear the container by writing the header block with ``container_state = 2``, ``consumed_attempt_counter``, and a fresh header CRC, followed by a flush. The regions are left intact as evidence for ``omarchy status`` until the OS next writes. The order BCR commit first, clear second makes a power loss between them idempotent (SM-17).

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
     - Mark board, manifest ID, derived payload digest, profile ID/digest, slot, slot generation, lineage, source generation, or F-07 receipt relation differs from the pending descriptor or trusted manifest
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
     - ``checks_digest`` or ``rollback_set_digest`` differs from U-Boot's recomputation; a required or allowed check is absent, duplicated, reordered, has the wrong class, measurement name, unit, bound, source evidence, failure limit, or is not ``pass``
     - ``BOOT_PROFILE_FAILURE`` or ``BOOT_REQUIRED_CHECK_FAILURE``; hold; clear
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
     - ``BOOT_FRESHNESS_FAILURE``; hold; clear. If BLK-06 is unavailable or mismatched, no counter-only substitute is allowed and the release path HALTs
   * - SM-22
     - ``Trusted<BootHealthCore>`` exists but the mark is absent, separately invalid, or not bound to that exact core
     - ``BOOT_MARKER_AUTH_FAILURE`` or ``BOOT_CONTEXT_MISMATCH``; no T-05; hold; clear
   * - SM-23
     - Mark is otherwise valid but the core was not constructed from the complete manifest-declared profile and source record
     - ``BOOT_PROFILE_FAILURE``; no T-05; hold; clear

A hold decision never consumes the last-known-good slot: the pending slot either continues its bounded budget (T-04) or fails (T-06, T-07, T-08), and only then is the last-known-good slot selected.

Release boot path
-----------------

Architecture and launcher decision
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The required release architecture is the program's ``m1n1 stage 1 -> m1n1 stage 2 + DT -> U-Boot -> GRUB -> kernel`` chain. This design chooses GRUB as the only release kernel launcher. Direct kernel boot from U-Boot (``booti``, ``bootm``, ``bootefi`` of a kernel EFI stub, or any bootflow method) is diagnostic and non-release: it is compiled out of the release profile, and even in a diagnostic profile it never installs a release BootContext, so it can never produce an accepted slot.

U-Boot's entry inputs from the opaque predecessor are exactly: a device-tree blob at the address the predecessor supplies; the root ``compatible`` and ``model`` values; ``/chosen`` properties that the existing board code already consumes (``asahi,efi-system-partition`` and ``/chosen/framebuffer``); and a memory map that does not overlap U-Boot, the DT, or the framebuffer. The human-signed predecessor envelope (BLK-10) must attest those inputs. U-Boot validates them at its boundary and makes no claim about how they are produced.

Sequence of the release command
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

0. Verify the exact ratified F-02/F-03 contract, generated bindings, authenticated opaque-predecessor interface envelope, device/installation binding, and independent monotonic-floor receipt. Missing or mismatched inputs produce ``HOLD/HALT`` before storage or slot access.
1. Resolve the board: read root ``compatible`` and ``model`` from the working device tree, match exactly one ``board-registry/v1`` board under ``identity_match.linux`` (delivery form BLK-11), and derive ``board_id``. Zero or more than one match is ``OMARCHY_BOARD_MISMATCH`` and HALT.
2. Resolve storage: gather all GPT disks and all partitions; resolve the Apple ESP by gathering every partition matching the exact ``asahi,efi-system-partition`` UUID and require exactly one match. Resolve BCR-0, BCR-1, and BSM by their complete typed identity tuples and require one live GPT disk GUID and parent/stable-ID relation for all of them. Zero or multiple ESP matches, duplicate UUIDs, or cross-disk tuples are ``BOOT_STORAGE_IDENTITY_FAILURE`` before any slot file is read. The current first-ESP fallback in ``asahi_esp_devpart()`` is excluded from the release profile (NI-13).
3. Read and select the BCR per `Two-copy selection and recovery`_; perform T-11 only when it is an authenticated same-disk repair and the independent floor permits it.
4. Consume and evaluate the BSM per `Mark evaluation cases`_; construct the trusted core first and the separate mark second; commit T-05 or T-07 only if both pass, then clear the container.
5. Stage detection: gather both ``slot-a`` and ``slot-b`` candidates that are not the exact current PENDING or LKG descriptors. Require exactly one F-07-authorized eligible candidate, including its promotion receipt, generation, lineage, rollback, board/profile, artifact, and anti-downgrade checks. Zero candidates means no staging; more than one is ``BOOT_CANDIDATE_CONFLICT`` with no PENDING write and ``HOLD/HALT``.
6. Select only the total state-machine outcomes: if ``selected_slot`` is exactly PENDING and ``attempts_used < attempt_limit``, reserve the next external counter and commit T-04; if its budget is exhausted, commit T-06 and re-select; if ``selected_slot`` is the exact LKG ACCEPTED descriptor, verify it and use T-09 on failure; if ``selected_slot`` is PINNED recovery, verify the immutable recovery identity and full recovery policy. FAILED, EMPTY, non-LKG ACCEPTED, unknown, and contradictory states are ``BOOT_STATE_CONSISTENCY_FAILURE`` and HALT.
7. Verify the slot: strict-parse and canonicalize the manifest, derive only ``sha256(JCS(payload))``, verify the signer and context, run the generated F-02 projection validator, verify the exact F-07 promotion receipt, then construct ``Trusted<PlatformManifest>``. Compare the derived digest to BCR-S08; check channel, board target, registry digest, consumer API, complete profile, anti-downgrade policy, and every declared artifact. On failure commit T-08 only when a valid transition is permitted; never replace a missing canonical dependency with recovery or a stable signature.
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

PROVISIONAL, pending coordinator approval: the sealed record fields are exactly ``context_schema = boot-context/v1``, ``board_id``, ``manifest_id``, ``manifest_digest``, ``lineage_id``, ``slot_id``, ``slot_generation``, ``attempt_counter``, ``source_generation``, ``atomic_record_digest``, ``lineage_source_digest``, ``device_installation_binding_digest``, ``monotonic_floor``, ``last_counter``, ``replay_reservation_digest``, ``permanent_tombstone_root``, and ``provenance`` with ``source_kind = atomic-boot-journal/v1``, ``source_api_version``, and ``storage_generation``. U-Boot fills them only from the committed canonical F-02 ``AtomicBootRecord``: ``atomic_record_digest = BCR-24 record_digest``, ``lineage_source_digest = BCR-25 authenticated_source_record_digest``, ``device_installation_binding_digest`` from the verified F-03 binding, ``monotonic_floor`` from the independent receipt, ``last_counter = BCR-27``, ``replay_reservation_digest = BCR-28``, ``permanent_tombstone_root = BCR-29``, and ``storage_generation`` is the canonical F-02 generation, never BCR-07 ``sequence``. The BootContext is not an independently signed payload; it is constructed only by the firmware-channel handoff after ``Trusted<AtomicBootRecord>`` and ``Trusted<BootContext>`` constructors succeed, and its authority is enforced by U-Boot's cross-check on the next boot.

Installation by U-Boot: after the T-04 commit, copy the control device tree, add the three properties, and pass the copy to ``efi_binary_run()``. ``efi_install_fdt()`` then copies it again and runs ``image_setup_libfdt()``; the ``bootargs`` fixup reads the environment, which in the release profile has no ``bootargs`` variable and no writable store, so the fixup adds nothing. The properties survive those fixups because ``ft_board_setup()`` and ``fdt_chosen()`` only add or replace their own named properties.

Preservation by GRUB: GRUB is required to forward the ``EFI_FDT_GUID`` table it received to the kernel's EFI stub, adding only its own ``/chosen`` properties for the initramfs and command line. This is the behavior expected of the GRUB arm64 EFI Linux loader when ``devicetree`` is not used, and it is exactly what experiment E-01 must prove for the pinned GRUB artifact before any implementation promotion. GRUB has no validation role and no ability to make a context trustworthy; it can only preserve or damage it.

Validation by the Linux consumer (K-01 owned): read the three properties from ``/proc/device-tree/chosen``; require ``boot-mode = release``; recompute and compare the SHA-256 property; strictly parse the record under the bounded boot binding; resolve the same exact physical disk and typed parent/stable-ID relation; read both BCR copies read-only, select by the same precedence, verify the complete F-02 journal fields and independent floor, and require BCR-24/BCR-25/BCR-27/BCR-28/BCR-29 to equal the context fields; require the descriptor tuple for ``slot_id`` to equal every record field; construct ``Trusted<AtomicBootRecord>`` through the generated F-02 constructor and then ``Trusted<BootContext>`` through the only constructor. Any failure is ``TRUST_BOUNDARY_FAILURE`` or the stable code from `Failure code/path/phase/result matrix`_, the health writer refuses to run, and no container write occurs.

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

HALT behavior is deterministic: print the local code string, both copies' census, the slot that was attempted, and the reason; then drop to the console with the banner ``OMARCHY RELEASE BOOT HALTED: manual actions are untrusted``. The release diagnostic allowlist is generated as ``release-command-allowlist/v1`` from the resolved release ``.config`` and the complete command-dispatch inventory. Its declared set is exactly ``omarchy``, ``reset``, and ``help``. It records every enabled command object, hidden command, alias, dynamic subcommand, and Kconfig-selected dispatch object; a command not in that set is ``RELEASE_COMMAND_CLOSURE_FAILURE`` and fails the build/design gate. The current tree has no resolved release ``.config`` or generated inventory, so this is an unimplemented gate, not a three-command result.

The generated inventory explicitly covers inherited ``font`` and ``smbios`` commands; generic boot, bootflow, bootmeth, EFI boot manager, direct EFI, direct kernel, removable-media, USB, NVMe, filesystem, serial-load, network, PXE, DHCP, media, environment, EFI-variable, script, hidden, alias, and dynamically registered commands. Each category must be absent from the resolved release dispatch inventory except for the explicitly allowed diagnostic set. The allowlist validator fails for any enabled command outside the set, including one introduced through a Kconfig default or a command object not named in the static table; enumeration order is never authority.

.. list-table::
   :header-rows: 1
   :widths: 28 26 20 26

   * - Inventory category
     - Required coverage
     - Example inherited surface
     - Result if enabled outside allowlist
   * - Explicit diagnostic
     - ``omarchy``, ``reset``, ``help`` only
     - ``CONFIG_CMD_OMARCHY`` and core reset/help dispatch
     - Allowed
   * - Inherited informational
     - Enumerate command registration and hidden symbols
     - ``font``, ``smbios``, aliases, dynamic subcommands
     - ``RELEASE_COMMAND_CLOSURE_FAILURE``; build fails
   * - Generic boot and media
     - Enumerate bootflow, bootmeth, EFI, kernel, USB, NVMe, filesystem, removable, and serial-load commands
     - ``BOOTSTD``, ``BOOTMETH_*``, ``CONFIG_CMD_BOOTEFI*``, ``CONFIG_CMD_BOOTI``, ``CONFIG_CMD_USB*``, ``CONFIG_CMD_LOAD*``
     - ``RELEASE_COMMAND_CLOSURE_FAILURE``; build fails
   * - Network and environment
     - Enumerate network, PXE, DHCP, script, save/import/export/edit/run/source, and EFI-variable dispatch
     - ``CONFIG_NO_NET``, ``CONFIG_CMD_*NET*``, ``CONFIG_CMD_SOURCE``, ``CONFIG_ENV_*``, ``CONFIG_EFI_*``
     - ``RELEASE_COMMAND_CLOSURE_FAILURE``; build fails

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
   * - ``CONFIG_CMD_SELECT_FONT``, ``CONFIG_CMD_SMBIOS``
     - ``n``
     - Explicitly removes the inherited ``font`` and ``smbios`` commands; the generated dispatch inventory must still verify their absence
   * - ``CONFIG_CMD_USB``, ``CONFIG_CMD_MMC``, ``CONFIG_CMD_SCSI``, ``CONFIG_CMD_DFU``, ``CONFIG_CMD_FASTBOOT``, all network/PXE/DHCP command symbols
     - ``n``
     - Removes removable, media, provisioning, and network command paths; any symbol not named here is caught by the exhaustive allowlist
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
   * - ``CONFIG_OMARCHY_RELEASE_COMMAND_ALLOWLIST``
     - ``y``
     - Requires the generated allowlist to equal the resolved dispatch inventory; any enabled command outside the diagnostic set fails the build/design gate

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
     - ``$.payload.document_id``, derived ``sha256(JCS($.payload))``, ``$.payload.channel``, ``$.payload.release_version``, and ``$.payload.promotion_receipt``
     - F-05 assembles; F-07 promotes; U-Boot consumes the exact receipt and derived digest
   * - Rollback predecessor
     - ``components.boot_stack.rollback.previous_manifest_ids`` and ``previous_component_ids``; top-level ``rollback`` projection with ``failure_attempt_limit`` and ``minimum_retention``
     - F-05; enforced by T-03
   * - Opaque predecessor envelope
     - No manifest path exists in the F-02 candidate for the human-signed stage-1/stage-2 artifact identity, interface attestation, license inventory, and provenance
     - BLK-10; the assembled predecessor payload that embeds ``u-boot-nodtb.bin`` is an F-05 assembly artifact whose role is also BLK-10

Typed provenance and promotion relations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Every predecessor interface, source identity, artifact role, recipe, toolchain, report, device binding, lineage, and promotion relation has the same typed handoff shape. A relation is not accepted from a path, branch, mutable reference, filename, or prose assertion. The following fields are mandatory in each future-ratified relation: producer, consumer and API/path, immutable preimage, artifact/document/content/payload digest, signer and context, freshness, first-failure code/path/phase/result, and blocked-until authority.

.. list-table::
   :header-rows: 1
   :widths: 14 11 15 18 17 15 12 22 14

   * - Relation
     - Producer
     - Consumer/API path
     - Immutable preimage
     - Digest binding
     - Signer/context
     - Freshness
     - Rejection code/path/phase/result
     - Blocked until
   * - predecessor interface
     - opaque predecessor owner
     - B-04 ``release.entry``
     - Exact typed interface tuple: DT address, root compatible/model, chosen inputs, memory map, and predecessor artifact role
     - document, content, and payload digests
     - owner-authorized F-03 envelope/context
     - cohort, issued-at, expiry, replay
     - ``BOOT_PROVENANCE_FAILURE`` at ``predecessor.interface`` during entry; ``REJECT/HALT``
     - BLK-10 and F-03
   * - source tree/archive identity
     - F-04 builder
     - B-03/F-05 ``source.identity``
     - One ratified source kind only: immutable git tree preimage or immutable source archive preimage; no branch, tag, ref, or mutable checkout
     - source content digest, document digest, and payload digest
     - F-04 builder binding and signing context
     - per candidate; commit/archive identity and expiry
     - ``BOOT_PROVENANCE_FAILURE`` at ``manifest.components.boot_stack.source`` during source admission; ``REJECT/HALT``
     - BLK-18 and F-04
   * - artifact role
     - F-05 assembler
     - U-Boot/GRUB artifact consumer
     - ``artifact_id``, role, kind, media type, size, content bytes, manifest ID, and source identity
     - artifact content digest, manifest document digest, and derived payload digest
     - F-05 manifest-release context and artifact policy
     - manifest expiry and generation; anti-replay
     - ``BOOT_PROVENANCE_FAILURE`` at ``manifest.components.*.artifacts[]`` during artifact admission; ``REJECT/HALT``
     - BLK-09 and F-05
   * - recipe/toolchain lock
     - F-04 builder
     - B-03/F-05 ``build.inputs``
     - Canonical recipe, compiler/binutils/DT compiler/container identities, flags, and ordered patch queue
     - content, document, artifact, and payload digests
     - F-04 builder attestation and build context
     - per candidate and two-builder comparison
     - ``BOOT_PROVENANCE_FAILURE`` at ``provenance.recipe/toolchain`` during build admission; ``REJECT/HALT``
     - BLK-16 and F-04
   * - report lock
     - F-04/B-03
     - F-05/F-07 ``reports[]``
     - Canonical report bytes, report kind, tool version, input digests, and result
     - report content/document/payload digests
     - report signer and exact CI/build context
     - per candidate; report expiry and input-generation equality
     - ``BOOT_PROVENANCE_FAILURE`` at ``provenance.report_lock`` during promotion; ``REJECT/HALT``
     - F-04 and B-03
   * - provenance aggregate
     - F-04/F-05
     - U-Boot ``manifest.provenance`` and F-07 closure
     - Canonical ordered relation set linking source, recipe, toolchain, report, artifact, license, and builder observations
     - aggregate document/content/payload digest plus each relation digest
     - F-04/F-05 attestation and exact promotion context
     - candidate generation, expiry, and anti-replay
     - ``BOOT_PROVENANCE_FAILURE`` at ``manifest.provenance`` during artifact admission; ``REJECT/HALT``
     - BLK-16, BLK-18, F-04, and F-05
   * - device-installation binding
     - F-02/F-03 authority
     - U-Boot ``storage.device_installation_binding``
     - Exact ``JCS(I)`` binding preimage in `Freshness and installation binding`_
     - binding document/content/payload digest and floor receipt digest
     - F-03 device authority/context
     - monotonic floor, issued-at, expiry, replay
     - ``BOOT_DEVICE_BINDING_FAILURE`` at ``storage.parent_relation`` during device binding; ``HOLD/HALT``
     - BLK-19
   * - F-07 promotion
     - F-07 promotion terminal
     - U-Boot ``manifest.admission.promotion_receipt``
     - Canonical receipt without signature, including candidate, rollback, full closure, board/profile/qualification, legal/public-ledger, channel, manifest, generation, lineage, health, signer/context, and anti-replay fields
     - receipt document/content/payload digest and manifest derived payload digest
     - F-07 owner-authorized signer and promotion context
     - issued-at, expiry, generation, lineage, replay reservation
     - ``BOOT_PROMOTION_FAILURE`` at ``manifest.payload.promotion_receipt`` during promotion admission; ``REJECT/HALT``
     - BLK-20 and F-07

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
     - Generated boot binding and manifest-authority validator for U-Boot; ``generated-output.lock`` entry with ``output_role = boot-binding`` and the exact F-02 projection guard (BLK-04)
     - Compiled lock equality
     - Parse, canonicalize, verify for the three boot-consumed types
     - Build fails
     - F-02 implementation owner
     - B-03 implementation admission
   * - DEP-03
     - F-03
     - B-03, B-04
     - ``Trusted<TrustContext>`` bundle with ``manifest-release``, ``boot-runtime``, device-binding, floor, and F-07 receipt authority bindings embedded in the U-Boot image (BLK-05)
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
     - Immutable ``platform-manifest/v1`` per slot with complete projection equality, embedded F-07 promotion receipt, generated GRUB configuration, hostile cross-repository fixtures, artifact IDs (BLK-09)
     - derived payload digest equality at every boot
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
     - Passing sandbox suite, E-01 evidence, hostile fixture census, exhaustive command allowlist, release profile ``.config``, and exact promotion-receipt closure
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
     - derived payload digest equality
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
     - Verified clock policy for boot consumers (BLK-06), ``DocumentId`` grammar (BLK-07), lineage allocation (BLK-08), ``boot_artifact_digest`` semantics (BLK-12), core envelope bound (BLK-13), fixed ``ExpectedContext`` IDs (BLK-14), device/floor authority (BLK-19), and F-07 receipt schema (BLK-20)
     - Ratification
     - Boot rows
     - Rows SM-21 and the affected fields stay BLOCKED
     - F-02
     - B-04 implementation admission

Exact NI/BLK dependency mapping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The NI and BLK rows above are the detailed consumer contracts; this compact index is the sole one-to-one dependency-row mapping used by the release and F-07 gates. Every ID below occurs exactly once in this mapping, and the gate fails on a missing or duplicate ID. The consumer, API/path, rejection code, handoff digest, verification command, owner, and due-before gate are the corresponding fields in the keyed NI or BLK row.

.. list-table::
   :header-rows: 1
   :widths: 14 34 52

   * - ID
     - Dependency/consumer row
     - Consumer
   * - NI-01
     - ``NI-01``
     - B-03 release profile and command-allowlist gate
   * - NI-02
     - ``NI-02``
     - B-04 U-Boot trusted consumer
   * - NI-03
     - ``NI-03``
     - B-04 CI fixture consumer
   * - NI-04
     - ``NI-04``
     - K-01/Linux and GRUB handoff consumer
   * - NI-05
     - ``NI-05``
     - B-04 hostile-fixture gate
   * - NI-06
     - ``NI-06``
     - B-03 pinned build gate
   * - NI-07
     - ``NI-07``
     - F-02 generated binding consumer
   * - NI-08
     - ``NI-08``
     - BCR/BSM durability consumer
   * - NI-09
     - ``NI-09``
     - F-02/F-03 freshness consumer
   * - NI-10
     - ``NI-10``
     - F-07 qualification consumer
   * - NI-11
     - ``NI-11``
     - F-04/F-05 provenance consumer
   * - NI-12
     - ``NI-12``
     - B-03 command-closure consumer
   * - NI-13
     - ``NI-13``
     - B-03 physical-storage consumer
   * - NI-14
     - ``NI-14``
     - I-04/K-01/P-05 handoff consumers
   * - BLK-01
     - ``BLK-01``
     - BCR partition storage consumer
   * - BLK-02
     - ``BLK-02``
     - BSM partition storage consumer
   * - BLK-03
     - ``BLK-03``
     - BCR authentication and key-custody consumer
   * - BLK-04
     - ``BLK-04``
     - generated boot-binding consumer
   * - BLK-05
     - ``BLK-05``
     - U-Boot trust-context consumer
   * - BLK-06
     - ``BLK-06``
     - core/mark freshness consumer
   * - BLK-07
     - ``BLK-07``
     - document-lineage consumer
   * - BLK-08
     - ``BLK-08``
     - lineage-ID consumer
   * - BLK-09
     - ``BLK-09``
     - artifact-role consumer
   * - BLK-10
     - ``BLK-10``
     - opaque predecessor interface consumer
   * - BLK-11
     - ``BLK-11``
     - board-registry consumer
   * - BLK-12
     - ``BLK-12``
     - boot-artifact digest consumer
   * - BLK-13
     - ``BLK-13``
     - BSM core-bound consumer
   * - BLK-14
     - ``BLK-14``
     - signature-context consumer
   * - BLK-15
     - ``BLK-15``
     - control-set storage binder
   * - BLK-16
     - ``BLK-16``
     - recipe/toolchain provenance consumer
   * - BLK-17
     - ``BLK-17``
     - immutable recovery consumer
   * - BLK-18
     - ``BLK-18``
     - source tree/archive consumer
   * - BLK-19
     - ``BLK-19``
     - device-binding and monotonic-floor consumer
   * - BLK-20
     - ``BLK-20``
     - F-07 promotion-receipt consumer

Hostile fixtures
----------------

Every fixture is a single mutation against an otherwise accepted state. The expected result is the stable ``(code, path, phase, result)`` tuple, the decision, and the component that must reject. These fixtures are required test content; none has run (NI-05).

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
     - C2-07; no write; immutable recovery identity plus full checks or HALT
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
     - T-03 refused because the derived payload digest equals the FAILED descriptor
   * - HF-29
     - ``missing-last-known-good``
     - PENDING slot exhausts its budget with ``last_known_good_slot = 0``
     - T-06 selects only the immutable recovery identity after full checks; otherwise HALT
   * - HF-30
     - ``alternate-stable-writer``
     - A manifest with ``channel = stable`` signed by a valid key whose binding lacks the ``manifest-release`` role, or a manifest without an F-07 promotion receipt
     - ``BOOT_PROMOTION_FAILURE`` at ``manifest.payload.promotion_receipt``; slot never PENDING; F-07 remains the only stable writer
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
     - Recovery slot manifest differs from the immutable compiled or independently authenticated identity/digest
     - ``BOOT_RECOVERY_IDENTITY_FAILURE``; full checks fail; HALT
   * - HF-34
     - ``full-disk-rollback``
     - Restore an older complete authenticated disk snapshot after a newer counter reservation or acceptance
     - ``BOOT_COUNTER_FAILURE`` at ``atomic_record.last_counter`` during freshness; ``HOLD/HALT``; no launch
   * - HF-35
     - ``full-disk-clone``
     - Clone the complete BCR/BSM/GPT tuple to another device of the same board class
     - ``BOOT_DEVICE_BINDING_FAILURE`` at ``storage.device_installation_binding`` during device binding; ``HOLD/HALT``; no slot read
   * - HF-36
     - ``manifest-projection-mismatch``
     - Alter only top-level ``artifacts``, ``package_set``, ``compatibility``, ``firmware_schema``, or ``rollback`` while component projections remain signed
     - ``BOOT_PROJECTION_FAILURE`` at ``manifest.payload.<projection>`` during generated F-02 validation; ``REJECT/HALT``
   * - HF-37
     - ``derived-digest-wire-injection``
     - Add a ninth ``payload_digest`` property to the envelope
     - ``PARSE_SCHEMA_FAILURE`` at ``envelope.payload_digest`` during strict parse; ``REJECT/HALT``; no derived value is read
   * - HF-38
     - ``derived-digest-mismatch``
     - Retain an old BCR-S08 while changing the canonical payload, or inject a stale external digest into the binding
     - ``BOOT_RECORD_DIGEST_FAILURE`` at ``manifest.derived_payload_digest`` during digest binding; ``REJECT/HALT``
   * - HF-39
     - ``invalid-check-class``
     - Use a signed check ID with a class outside the manifest-declared allowed class set
     - ``BOOT_PROFILE_FAILURE`` at ``boot_health.checks[].class`` during core admission; ``HOLD``; no T-05
   * - HF-40
     - ``invalid-measurement``
     - Use a wrong measurement name/unit or a value outside the declared bound
     - ``BOOT_PROFILE_FAILURE`` at ``boot_health.checks[].measurement`` during core admission; ``HOLD``; no mark acceptance
   * - HF-41
     - ``profile-limit-or-source-mismatch``
     - Change ``failure_limit``, source evidence/record, profile digest, or exact check order while retaining passing statuses
     - ``BOOT_PROFILE_FAILURE`` at ``boot_health.profile.<field>`` during core admission; ``HOLD``; no trusted core
   * - HF-42
     - ``bcr-loss``
     - Erase or corrupt both BCR copies so no committed record or floor-bound state is available
     - ``BOOT_RECORD_SOURCE_FAILURE`` at ``bcr.atomic_record`` during copy selection; ``HOLD/HALT``; immutable recovery only
   * - HF-43
     - ``missing-f07-receipt``
     - Present a correctly signed stable-channel manifest without the F-07 promotion receipt
     - ``BOOT_PROMOTION_FAILURE`` at ``manifest.payload.promotion_receipt`` during manifest admission; ``REJECT/HALT``; no PENDING
   * - HF-44
     - ``inherited-command``
     - Leave inherited ``font`` or ``smbios`` dispatch enabled, or introduce a hidden/alias/dynamic command outside the diagnostic set
     - ``RELEASE_COMMAND_CLOSURE_FAILURE`` at ``release_command_allowlist.enabled_command`` during the build gate; ``REJECT/HALT``; no release image
   * - HF-45
     - ``selected-state-cartesian``
     - Exercise every one of the 2,500 selected-slot, descriptor, LKG, other-slot, and recovery-state combinations
     - ``BOOT_STATE_CONSISTENCY_FAILURE`` at ``atomic_record.selected_slot`` during state validation for every illegal tuple; ``HOLD/HALT``; no write
   * - HF-46
     - ``divergent-recovery``
     - Make equal-counter valid BCR copies disagree on the recovery identity, policy, or selected state
     - ``OMARCHY_BCR_DIVERGENT`` at ``bcr.copy[0|1]`` during copy selection; no write and no record selection; immutable recovery only or HALT
   * - HF-47
     - ``cross-disk-tuple``
     - Put a valid ESP on one disk and valid BCR/BSM records on another disk with individually matching local GUIDs
     - ``BOOT_STORAGE_IDENTITY_FAILURE`` at ``storage.parent_relation`` during physical resolution; ``HOLD/HALT``; no slot read
   * - HF-48
     - ``duplicate-esp-uuid``
     - Put the exact ESP UUID on two partitions or two NVMe namespaces with different slot content
     - ``BOOT_STORAGE_IDENTITY_FAILURE`` at ``storage.esp.matches`` during exact-one resolution; ``HOLD/HALT``; enumeration order is ignored
   * - HF-49
     - ``dual-candidate``
     - Place two distinct F-07-authorized eligible stable candidates in the two non-current slots
     - ``BOOT_CANDIDATE_CONFLICT`` at ``candidate_set`` during stage detection; ``HOLD/HALT``; no PENDING write
   * - HF-50
     - ``document-id-reuse``
     - Re-stage a prior document ID with a different payload digest after its descriptor was recycled
     - ``DOCUMENT_ID_REUSE`` at ``atomic_record.permanent_tombstones`` during lineage admission; ``REJECT/HALT``; no staging
   * - HF-51
     - ``document-id-fork``
     - Present two generations with one document ID and divergent payload digests in the durable history
     - ``DOCUMENT_ID_FORK`` at ``atomic_record.permanent_tombstones`` during lineage admission; ``REJECT/HALT``; no staging

Empirical residuals
-------------------

These are NOT IMPLEMENTED gates observed at the current tip. They are never design accomplishments. The rows below are also the NI dependency/consumer rows: each NI ID appears exactly once with its consumer, consumer API/path, rejection code, handoff artifact/digest, verification command, owner, and due-before gate. Missing or duplicate mapping blocks release and F-07 promotion.

.. list-table::
   :header-rows: 1
   :widths: 7 25 13 18 17 20 22 13 22

   * - ID
     - Observation
     - Consumer
     - Consumer API/path
     - Rejection code
     - Handoff artifact/digest
     - Verification command
     - Owner
     - Due before
   * - NI-01
     - ``configs/apple_m1_defconfig`` still uses ``CONFIG_BOOTCOMMAND="bootflow scan -b"``; ``configs/apple_omarchy_release_defconfig`` does not exist
     - B-03 release gate
     - ``configs/apple_omarchy_release_defconfig``; resolved ``.config``
     - ``RELEASE_COMMAND_CLOSURE_FAILURE``
     - ``config-file/v1`` and allowlist digest
     - pinned-builder config resolve plus generated allowlist check
     - B-03
     - B-03 implementation admission
   * - NI-02
     - No Omarchy command, Kconfig symbol, schema, binding, state writer, or validator exists in this tree; ``cmd/omarchy.c``, ``boot/omarchy_slot.c``, ``schemas/platform-manifest.json``, and ``schemas/boot-health.json`` are absent, and the last two must never exist here because schemas are published only by ``omarchy-apple-platform``
     - B-04 U-Boot consumer
     - ``cmd/omarchy.c`` and ``boot/omarchy_slot.c``
     - ``TRUST_BOUNDARY_FAILURE``
     - generated boot-binding output digest
     - pinned U-Boot build and binding constructor test
     - B-04
     - B-04 implementation admission
   * - NI-03
     - No unit or Python test for any Omarchy contract exists; ``test/dm/test_omarchy_slot.c`` and ``test/py/tests/test_omarchy_slots.py`` are absent
     - B-04 CI gate
     - ``test/dm/test_omarchy_slot.c`` and ``test/py/tests/test_omarchy_slots.py``
     - ``CI_CONFORMANCE_FAILURE``
     - test-report digest covering C2, SM, T, HF
     - ``./test/py/test.py test/py/tests/test_omarchy_slots.py`` and sandbox suite
     - B-04
     - B-04 implementation admission
   * - NI-04
     - The success-mark and GRUB preservation experiment E-01 has not run; ``test/py/tests/test_omarchy_boot_context.py`` is absent
     - K-01/Linux and GRUB handoff
     - ``test/py/tests/test_omarchy_boot_context.py``; E-01
     - ``BOOT_CONTEXT_MISMATCH``
     - E-01 log, disk-image, and context digest bundle
     - pinned GRUB/U-Boot E-01 experiment
     - B-04
     - B-04 implementation promotion
   * - NI-05
     - No hostile fixture in `Hostile fixtures`_ (HF-01 through HF-51) has been executed
     - B-04 hostile-fixture gate
     - HF-01 through HF-51 fixture inventory
     - first row-specific code in `Failure code/path/phase/result matrix`_
     - hostile-fixture manifest and result digest
     - generated hostile corpus runner; require 0 unobserved rows
     - B-04
     - B-04 implementation promotion
   * - NI-06
     - Local configuration and build validation was unavailable: host GNU Make 3.81 fails at ``Makefile:247``; no ``.config`` was resolved and no binary was produced
     - B-03 release build gate
     - pinned builder and resolved ``.config``
     - ``BOOT_PROVENANCE_FAILURE``
     - two-builder build/report lock digest
     - pinned builder config, build, and diff report
     - B-03
     - B-03 implementation admission on a pinned builder
   * - NI-07
     - No Ed25519 verifier and no JSON or RFC 8785 canonicalizer exists under ``lib/``; the boot binding must supply them within the bounded limits
     - generated F-02 binding
     - strict parser, JCS, Ed25519, and ``Trusted<T>`` constructors
     - ``TRUST_BOUNDARY_FAILURE``
     - ``generated-output.lock`` boot-binding digest
     - binding unit corpus plus bounded-memory build
     - B-04 with DEP-02
     - B-04 implementation admission
   * - NI-08
     - The block layer has no flush operation and the NVMe driver never issues ``nvme_cmd_flush``; the commit procedure's flush steps cannot be honored until a flush operation exists and is proven on Apple NVMe after power removal
     - BCR writer
     - block flush/barrier API at each BCR/BSM durable write
     - ``BOOT_RECORD_COMMIT_FAILURE``
     - storage durability report digest
     - power-cut sandbox and Apple NVMe qualification
     - B-04
     - B-04 implementation admission and Q-04 power-loss rows
   * - NI-09
     - No verified clock source exists in U-Boot on Apple hardware; SM-21 is governed by BLK-06
     - freshness validator
     - ``boot_health.freshness`` and ``success_mark.freshness``
     - ``BOOT_FRESHNESS_FAILURE``
     - signed clock/floor policy digest
     - ratified-clock and replay fixture suite
     - F-02 and F-03
     - B-04 implementation admission
   * - NI-10
     - No physical qualification evidence exists for any board; nothing in this document is a support or compatibility claim
     - F-07 qualification gate
     - Q-04 through Q-08 board/profile records
     - ``QUALIFICATION_FAILURE``
     - signed qualification-record and public-ledger digests
     - disposable qualified hardware matrix
     - Hardware lab
     - Q-04 through Q-08
   * - NI-11
     - No exact release configuration record, two-builder comparison, SBOM, or provenance exists for ``boot-stack``
     - F-04/F-05 assembler
     - ``manifest.provenance`` and ``report_lock``
     - ``BOOT_PROVENANCE_FAILURE``
     - SBOM, source, recipe, toolchain, and report-lock digests
     - F-04 provenance and F-05 assembly checks
     - F-04 and B-03
     - F-05 assembly
   * - NI-12
     - ``ENV_IS_IN_FAT``, ``EFI_VARIABLE_FILE_STORE``, ``EFI_BOOTMGR``, ``BOOTSTD``, and the EFI bootmeths remain enabled by default for the Apple baseline; the release exclusions are design only
     - release command allowlist
     - resolved ``.config`` and dispatch inventory
     - ``RELEASE_COMMAND_CLOSURE_FAILURE``
     - exhaustive command-allowlist digest
     - config lint plus command registration inventory
     - B-03
     - B-03 implementation admission
   * - NI-13
     - ``asahi_esp_devpart()`` falls back to the first EFI system partition when the UUID is absent or unmatched; the release rule of UUID-only resolution is not implemented
     - physical storage resolver
     - ``storage.esp.matches`` and same-disk parent relation
     - ``BOOT_STORAGE_IDENTITY_FAILURE``
     - GPT census and resolver report digest
     - GPT duplicate/zero/cross-disk fixture suite
     - B-03
     - B-03 implementation admission
   * - NI-14
     - The T-01 provisioning writer, the OS BSM writer, and the Linux BootContext consumer do not exist in any repository
     - installer, K-01, and P-05 consumers
     - T-01 writer, BSM writer, and ``Trusted<AtomicBootRecord>`` adapter
     - ``TRUST_BOUNDARY_FAILURE``
     - install/BootContext/BSM handoff digest bundle
     - installer, Linux consumer, and BSM integration suites
     - I-04, K-01, P-05
     - B-04 implementation promotion

BLOCKED external constants
--------------------------

Each constant is owned outside this lane. Until ratified, U-Boot has no value for it, cannot be built for release, and B-03/B-04 admission is BLOCKED. No value is guessed here. The rows below are also the BLK dependency/consumer rows: each BLK ID appears exactly once with its consumer, consumer API/path, rejection code, handoff artifact/digest, verification command, owner, and due-before gate. Missing or duplicate mapping blocks release and F-07 promotion.

.. list-table::
   :header-rows: 1
   :widths: 8 24 13 18 17 20 22 13 22

   * - ID
     - Constant
     - Consumer
     - Consumer API/path
     - Rejection code
     - Handoff artifact/digest
     - Verification command
     - Owner
     - Due before
   * - BLK-01
     - ``OMARCHY_BCR_PARTITION_TYPE_GUID`` for the two boot-control partitions in the typed storage registry
     - storage resolver
     - ``storage.bcr[0|1].type_guid``
     - ``BOOT_STORAGE_IDENTITY_FAILURE``
     - typed-storage record and GPT census digest
     - GPT identity resolver test
     - F-02 with I-03
     - B-04 implementation admission
   * - BLK-02
     - ``OMARCHY_BSM_PARTITION_TYPE_GUID`` for the success-mark partition
     - BSM resolver
     - ``storage.bsm.type_guid``
     - ``BOOT_STORAGE_IDENTITY_FAILURE``
     - typed-storage record and BSM census digest
     - GPT identity resolver test
     - F-02 with I-03
     - B-04 implementation admission
   * - BLK-03
     - ``OMARCHY_BCR_AUTH_KEY`` source, provisioning, and rotation; the key must be unavailable to the Linux OS at runtime. No absence of a key source permits algorithm 1 in release; it is a HOLD/HALT threat-model decision
     - BCR verifier/writer
     - ``bcr.auth_tag`` and F-03 key binding
     - ``TRUST_BOUNDARY_FAILURE``
     - key-custody and trust-context digest
     - key-custody review and authenticated-record fixtures
     - F-03 with the human predecessor owner
     - B-04 release profile gate
   * - BLK-04
     - Boot binding target for U-Boot: the candidate names ``rust-boot`` outputs with ``output_role = boot-binding`` while U-Boot is a C code base; the ratified language, linkage, and bounded-memory report for the U-Boot consumer
     - generated binding consumer
     - ``generated-output.lock``
     - ``BINDING_INTEGRITY_FAILURE``
     - boot-binding document/content/payload digest
     - generated binding build and constructor tests
     - F-02
     - B-03 implementation admission
   * - BLK-05
     - Embedded ``Trusted<TrustContext>`` bundle format, offline expiry handling, and rotation procedure for a firmware consumer without network
     - all authenticated U-Boot consumers
     - ``Trusted<TrustContext>`` constructor
     - ``TRUST_BOUNDARY_FAILURE``
     - trust-bundle document/content/payload digest
     - trust-context expiry, revocation, and role tests
     - F-03
     - B-04 release profile gate
   * - BLK-06
     - Verified clock and freshness policy for boot consumers; counter equality is not a substitute for a ratified clock/floor policy
     - core/mark freshness validator
     - ``boot_health.freshness`` and ``success_mark.freshness``
     - ``BOOT_FRESHNESS_FAILURE``
     - signed clock/floor policy digest
     - freshness and replay test suite
     - F-02 and F-03
     - B-04 implementation admission
   * - BLK-07
     - ``DocumentId`` grammar; BCR-S09 stores its SHA-256 so the record width is independent of the ruling
     - lineage validator
     - ``atomic_record.permanent_tombstones``
     - ``DOCUMENT_ID_REUSE`` or ``DOCUMENT_ID_FORK``
     - canonical document-lineage digest
     - document-ID grammar and fork fixtures
     - F-02
     - B-04 implementation admission
   * - BLK-08
     - ``lineage_id`` allocation rule and its exact authenticated source; no local deterministic derivation is accepted until F-02 ratifies the rule
     - lineage validator
     - ``atomic_record.lineage_id`` and tombstone history
     - ``BOOT_COUNTER_FAILURE`` or ``DOCUMENT_ID_FORK``
     - lineage document/content/payload digest
     - lineage allocation and reuse fixtures
     - F-02
     - B-04 implementation admission
   * - BLK-09
     - Artifact IDs and media types for the U-Boot image, GRUB image, GRUB configuration, kernel image, initramfs, DTB set, and recovery payload
     - manifest/artifact consumer
     - ``manifest.components.*.artifacts[]``
     - ``BOOT_PROVENANCE_FAILURE`` or ``OMARCHY_ARTIFACT_DIGEST_MISMATCH``
     - artifact ID, content, document, and derived payload digests
     - artifact-role and digest fixture suite
     - F-05 with F-04
     - B-04 implementation admission
   * - BLK-10
     - Manifest path and interface attestation schema for the opaque predecessor envelope, and the role of the assembled predecessor payload that embeds ``u-boot-nodtb.bin``
     - U-Boot entry consumer
     - ``predecessor.interface``
     - ``BOOT_PROVENANCE_FAILURE``
     - opaque envelope document/content/payload digest
     - signed-interface verification using opaque inputs only
     - F-02 with the human predecessor owner (B-01)
     - B-03 implementation admission
   * - BLK-11
     - Delivery form of the ``board-registry/v1`` document to U-Boot (embedded at build or staged in the slot) and the exact ``identity_match.linux`` matching rule
     - board resolver
     - ``board_registry`` and ``identity_match.linux``
     - ``OMARCHY_BOARD_MISMATCH``
     - board-registry document/content/payload digest
     - exact-one board matching test
     - F-02 with Q-00
     - B-03 implementation admission
   * - BLK-12
     - Semantics of ``slot.boot_artifact_digest``; U-Boot proposes the GRUB image ``content_digest``
     - slot/artifact verifier
     - ``bcr.slot.boot_artifact_digest``
     - ``OMARCHY_ARTIFACT_DIGEST_MISMATCH``
     - artifact content and manifest derived payload digests
     - artifact digest and role binding test
     - F-02
     - B-04 implementation admission
   * - BLK-13
     - Complete ``boot-health/v1`` core envelope byte maximum; U-Boot reserves 4,096 bytes
     - BSM/core reader
     - ``bsm.core_region``
     - ``RESOURCE_LIMIT``
     - core document/content/payload digest
     - bounded envelope parser test
     - F-02
     - B-04 implementation admission
   * - BLK-14
     - Fixed ``project_id``, ``repository_id``, and ``slice_id`` values in ``ExpectedContext`` for the ``boot-health-core`` and ``boot-success-marker`` rows
     - signature-context verifier
     - ``ExpectedContext`` for core and mark
     - ``SIGNATURE_CONTEXT_MISMATCH``
     - signing-row document/content/payload digest
     - exact-context hostile fixture suite
     - F-02 with F-03
     - B-04 implementation admission
   * - BLK-15
     - ``control_set_id`` derivation from the installer plan target identity
     - BCR/BSM storage binder
     - ``bcr.control_set_id`` and ``bsm.control_set_id``
     - ``BOOT_DEVICE_BINDING_FAILURE``
     - installer-plan target document/content/payload digest
     - install identity and transplant test
     - I-03
     - B-04 implementation admission
   * - BLK-16
     - Toolchain IDs, versions, and builder container digest for ``boot-stack``
     - provenance and build gate
     - ``provenance.recipe/toolchain``
     - ``BOOT_PROVENANCE_FAILURE``
     - toolchain, recipe, artifact, and report-lock digests
     - two-builder provenance comparison
     - F-04
     - F-05 assembly
   * - BLK-17
     - Recovery slot manifest policy: channel, signer, retention, immutable identity/digest refresh, and anti-downgrade rules; no board-plus-signature or hidden emergency fallback
     - recovery verifier
     - ``recovery.immutable_identity`` and recovery manifest
     - ``BOOT_RECOVERY_IDENTITY_FAILURE``
     - recovery identity, manifest, artifact, and policy digests
     - recovery replacement and lost-BCR fixtures
     - F-05 with F-07
     - B-04 implementation admission
   * - BLK-18
     - Exact source identity preimage: one ratified git tree object or one ratified source archive format; the alternatives are typed and mutually exclusive
     - source/provenance verifier
     - ``manifest.components.boot_stack.source``
     - ``BOOT_PROVENANCE_FAILURE``
     - source tree/archive, document, content, and payload digests
     - source identity and substitution fixtures
     - F-02 with F-04
     - F-05 assembly
   * - BLK-19
     - Independent monotonic per-device/installation floor and authenticated current-device/installation identity, including exact preimages and advance/reservation authority
     - freshness/device binder
     - ``storage.device_installation_binding`` and ``atomic_record.last_counter``
     - ``BOOT_DEVICE_BINDING_FAILURE`` or ``BOOT_COUNTER_FAILURE``
     - binding document/content/payload digest and floor receipt digest
     - floor rollback, clone, reservation, and power-loss fixtures
     - F-02/F-03 with I-03
     - B-04 implementation admission
   * - BLK-20
     - Exact F-07 promotion receipt/attestation schema, signer custody, full closure, and anti-replay/freshness policy
     - manifest admission
     - ``manifest.payload.promotion_receipt``
     - ``BOOT_PROMOTION_FAILURE``
     - receipt document/content/payload digest and manifest derived payload digest
     - missing, stale, forked, and incomplete receipt fixtures
     - F-07 with F-02/F-03
     - F-07 promotion

Acceptance and power-loss test plan
-----------------------------------

The first implementation extends the existing surfaces rather than claiming they cover the contract: ``test/dm/fwu_mdata.c`` for the pattern of two-copy metadata tests, ``test/py/tests/test_gpt.py`` for identity-tuple resolution and rejection of an ambiguous layout, ``test/boot/bootflow.c`` only to prove that no bootflow exists in the release profile, ``test/py/tests/test_efi_bootmgr.py`` only to prove that ``BootOrder`` cannot select anything, and ``test/py/tests/test_distro.py`` as a console-interaction reference. New sandbox tests use a disposable host-bound disk image and inject a failure at every durable write boundary; after every simulated reset the harness captures both BCR copies, the BSM header, the slot directory listing, the selected slot, the attempt counter, and the decision string.

The B-04 implementation gate is: all 14 C2 rows, all 23 SM rows, all 12 T rows, and all 51 HF rows have deterministic sandbox results; E-01 passes; the release ``.config`` resolves on a pinned builder with the two-builder comparison and exhaustive command inventory; every NI/BLK row has exactly one consumer mapping; and all 20 BLK constants are ratified. The physical gate additionally requires the Q-04 rows on disposable qualified hardware with rehearsed outer recovery. A green sandbox run, a U-Boot prompt, a recognized SoC, or a booting desktop is never qualification evidence.

Closing statement
-----------------

This document is design only. It contains no marker and no unresolved local design question: every external authority still open is a named BLOCKED constant with an owner and a due-before gate, and every implementation or evidence gap is a NOT IMPLEMENTED residual with a per-item consumer contract. F-02 and F-03 remain unratified for this consumer, B-03 and B-04 remain open and not started, their implementation admission remains BLOCKED, no build succeeded, no fixture or experiment ran, no hardware booted, and nothing here is DONE.
