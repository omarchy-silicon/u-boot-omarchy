.. SPDX-License-Identifier: GPL-2.0+

Omarchy Silicon release and boot-slot design
============================================

Status and scope
----------------

This is the B-03/B-04 design note for ``omarchy-silicon/u-boot-omarchy``, final bounded design-correction round 3. It is a design-only document. It is not an implementation, a build result, a schema, a binding, a test result, a compatibility claim, a support claim, a qualification record, a release approval, or a recovery claim. Nothing described here is DONE. B-03 and B-04 remain open, not-started slices in the canonical program ledger, and their implementation admission is BLOCKED until the upstream contracts named in this document are ratified.

The canonical program is ``omarchy-apple-platform/PROGRAM.md`` at the ratified commit ``58302d148f0e8b855578f9aa518ff1c5eb48c515``. This document binds U-Boot to the frozen cross-repository contracts listed in that program and creates no second authority.

Every wire detail derived from the F-02 candidate ``docs/design/platform-schema.md`` at ``c315c7e79928d0041deb582bed79a61074361b21`` is PROVISIONAL. That candidate is REJECTED and frozen pending the owner checkpoint recorded in the program. Provisional details are quoted so that the U-Boot binding is exact rather than illustrative; they are not a local shadow schema, they are not implemented here, and they do not become authority by appearing in this document. Where an F-02- or F-03-owned identifier is not ratified, this document uses a named BLOCKED constant from the table in `BLOCKED external constants`_ instead of a guessed value.

The coordinator has fenced ``m1n1-omarchy`` and every m1n1 path as an opaque human-produced boundary. This document does not inspect, read, analyze, edit, test, clone, fetch, browse, traverse, characterize, or make claims about that repository or its outputs. The predecessor stage appears here only as a coordinator-supplied signed opaque artifact envelope and as the observable inputs U-Boot consumes at its own entry point.

Ratification and fail-closed gate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The current text is a design proposal only. The F-02 and F-03 documents, generated bindings, authority bindings, monotonic-floor authority, device/installation identity authority, and their digests are external inputs, not values defined by this file. Before any ``Trusted<T>`` constructor, slot read, recovery launch, state transition, or release build, U-Boot must compare the locally compiled contract identity with the exact ratified F-02/F-03 identities and verify the authenticated source records. A missing, expired, revoked, unavailable, or mismatched canonical dependency has exactly one result: ``BINDING_INTEGRITY_FAILURE`` at ``contract.authority_lock`` during ratification, terminal action ``TA-HOLD-NO-WRITE``. No constructor, storage write, selection, recovery launch, or release admission occurs. A local alias, guessed value, stable-channel signature, copied disk value, or opaque predecessor output cannot satisfy this gate.

Future F-02/F-03 identity lock
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ratification imports exactly one ``contract-authority-import/v1`` object with exactly four members: ``import_schema``, ``f02``, ``f03``, and ``schema``. ``import_schema`` is the fixed string ``contract-authority-import/v1``. ``f02`` and ``f03`` are ``contract-authority-lock/v1`` objects, and ``schema`` is a ``schema-authority-lock/v1`` object. The import is an external F-02/F-03 handoff, not a local authority record.

Each ``contract-authority-lock/v1`` member has exactly ``lock_schema``, ``contract_id``, ``owner_id``, ``repository_id``, ``document_id``, ``source_commit``, ``content_digest``, ``payload_digest``, ``schema_set_digest``, ``generated_binding_digest``, ``authority_binding_digest``, and ``ratification_receipt_digest``. The ``schema-authority-lock/v1`` member has exactly ``lock_schema``, ``schema_set_id``, ``schema_set_version``, ``repository_id``, ``document_id``, ``source_commit``, ``content_digest``, ``payload_digest``, ``schema_set_digest``, ``generated_binding_digest``, and ``ratification_receipt_digest``. ``source_commit`` is an immutable commit identity; a branch, tag, ref, local alias, or copied object is not an identity. The release image embeds all three canonical lock objects byte-for-byte. The runtime authority input must carry the same three locks and their authenticated source records; U-Boot compares the fields in object order (import, F-02, F-03, schema), compares every cross-lock ``schema_set_digest`` and ``generated_binding_digest`` relation, verifies every ratification receipt, and only then constructs the imported F-02/F-03 types. No value in this paragraph supplies an authority value.

Strict parsing rejects an unknown or duplicate lock field as ``PARSE_SCHEMA_FAILURE`` before an identity object exists. After strict parsing, any missing, expired, revoked, unavailable, or mismatched import member, lock field, source record, schema-set digest, generated binding, authority binding, or ratification receipt has exactly one result: ``BINDING_INTEGRITY_FAILURE`` at ``contract.authority_lock`` during ratification, result ``HOLD``, terminal action ``TA-HOLD-NO-WRITE``. This is the only missing/mismatch result for F-02, F-03, or schema authority. No ``TRUST_BOUNDARY_FAILURE``, freshness result, recovery result, or local alias may replace it. The unresolved locks therefore keep implementation admission in HOLD; they never become local authority by being copied into this design note.

The authority-lock guard has four named mutations: remove ``f02``, remove ``f03``, change ``schema.schema_set_digest``, and change ``f03.authority_binding_digest``. Each mutation is rejected at the same exact tuple ``(BINDING_INTEGRITY_FAILURE, contract.authority_lock, ratification, HOLD)`` with terminal action ``TA-HOLD-NO-WRITE``. A valid lock copied from another branch, repository, document, schema set, or generation is a mismatch at this same tuple; no fixture may substitute a local value or choose a later phase.

The proposal becomes eligible for implementation only after the coordinator ratifies the exact record, manifest, profile, promotion, storage, and authority relations below. Ratification does not imply implementation, CI, power-cut evidence, physical qualification, release approval, or DONE status.

Verdict on the previous tip
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The coordinator rejected tip ``e385874ca1987248b40aa150f10d3d3ad32fde02`` with eight contract failures: unclosed cross-contract authority, a conceptual boot-control record, conflated success authority, an unclosed GRUB handoff, unenforced release selection, illustrative artifact roles, a missing toctree entry, and zero executable artifacts. This round corrects only the six bounded design seams named by the latest gate; none is addressed by implementation, because this lane owns only this document.

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

``ExpectedContext`` is constructed by U-Boot for each verification with the exact row from the eight-row signing contract and the separately ratified F-07 receipt context: ``payload_type``, ``payload_version``, ``domain``, ``context``, ``project_id``, ``repository_id``, ``slice_id``, ``operation``, ``board_id``, ``manifest_id``, ``manifest_digest``, ``schema_set_digest``, ``target_account_id``, ``target_account_binding``, ``target_identity_digests``, and ``policy_digest``. The fixed ``project_id``, ``repository_id``, and ``slice_id`` values for the boot rows are BLK-14. A mismatch on a signing-row member is ``SIGNATURE_CONTEXT_MISMATCH``; a receipt member mismatch is ``BOOT_PROMOTION_FAILURE``.

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

The only accepted manifest construction is ``strict_parse -> canonicalize -> derive_payload_digest -> envelope and signer verification -> generated F-02 projection validation -> F-07 promotion-receipt validation -> Trusted<PlatformManifest>``. The projection validator is not a best-effort warning and is not a consumer-local alias. A missing or mismatched generated validator, schema-set digest, or source record returns ``BINDING_INTEGRITY_FAILURE`` at ``contract.authority_lock``. A projection mismatch returns ``BOOT_PROJECTION_FAILURE`` at ``manifest.projection``. In both cases the manifest is absent. Stable-channel signature verification without this predecessor never constructs the trusted type.

The F-07 promotion receipt is a separately authenticated, owner-authorized relation embedded at ``$.payload.promotion_receipt`` and bound into the canonical manifest. It is not a ninth top-level payload type or a consumer-local alias. Its exact future-ratified object must contain ``receipt_schema``, ``candidate_digest``, ``rollback_digest``, ``closure_digest``, the complete required-slice closure, ``board_id``, ``profile_digest``, ``qualification_record_digest``, ``legal_digest``, ``public_ledger_digest``, ``channel``, ``manifest_id``, ``manifest_digest``, ``generation``, ``lineage_id``, ``health_evidence_digest``, ``signer_id``, ``signing_context``, ``issued_at``, ``expires_at``, ``replay_id``, and ``receipt_signature``. U-Boot verifies every field, the receipt signer and context under ``f07-promotion``, freshness, anti-replay, and equality to the manifest and current record. Missing, stale, forked, incomplete, or mismatched receipt is ``BOOT_PROMOTION_FAILURE`` at ``manifest.admission.promotion_receipt`` with result ``REJECT``. A stable-channel signature alone is never sufficient.

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
     - ``manifest.projection.artifacts``; ``REJECT``
   * - 2
     - ``$.payload.package_set`` against the recomputed package projection; exact length, key, order, and every field
     - ``BOOT_PROJECTION_FAILURE``
     - ``manifest.projection.package_set``; ``REJECT``
   * - 3
     - ``$.payload.compatibility`` against the recomputed compatibility projection; exact length, key, order, and every field
     - ``BOOT_PROJECTION_FAILURE``
     - ``manifest.projection.compatibility``; ``REJECT``
   * - 4
     - ``$.payload.firmware_schema`` against the recomputed firmware-schema projection; exact length, key, order, and every field
     - ``BOOT_PROJECTION_FAILURE``
     - ``manifest.projection.firmware_schema``; ``REJECT``
   * - 5
     - ``$.payload.rollback`` against the recomputed rollback projection; exact length, key, order, and every field
     - ``BOOT_PROJECTION_FAILURE``
     - ``manifest.projection.rollback``; ``REJECT``
   * - 6
     - ``$.payload.promotion_receipt`` complete relation and exact equality to manifest, record, board, profile, and qualification inputs
     - ``BOOT_PROMOTION_FAILURE``
     - ``manifest.admission.promotion_receipt``; ``REJECT``

Boot-health profile binding
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before ``Trusted<BootHealthCore>`` exists, U-Boot must bind the complete manifest-declared BootCheck profile. The bound profile is the exact allowed class set, check ID set and order, measurement name, unit, lower and upper bounds for every check, retry limit, failure limit, source evidence and source record identity, status vocabulary, profile digest, and freshness interval. The core's checks must have exactly the declared set and order, each ID exactly once, each class allowed, each measurement name and unit exact, each measurement within its declared bounds, each source record equal to the declared evidence, and each status equal to the closed vocabulary. ``checks_digest`` is recomputed only after those checks; it cannot launder an invalid profile.

The core path is ``strict_parse -> canonicalize -> verify boot-runtime -> compare profile identity and digest -> compare allowed classes and exact check set/order -> compare measurement names/units/bounds -> compare retry/failure limits -> compare source evidence/record -> verify freshness -> Trusted<BootHealthCore>``. A profile-field failure returns ``BOOT_PROFILE_FAILURE``; a check-entry failure returns ``BOOT_REQUIRED_CHECK_FAILURE``. Each has the first failing canonical check path and no trusted core. Success-mark validation is a separate later path: ``Trusted<BootHealthCore> -> strict_parse/verify boot-success-mark -> exact core, pending descriptor, counter, freshness, replay, and rollback comparisons -> T-05``. A valid core never implies a valid mark, and a valid mark never constructs a core.

Freshness is checked against the ratified F-02/F-03 clock and monotonic policy at both core and mark admission. Until that authority is ratified, the result is ``BOOT_FRESHNESS_FAILURE`` with result ``HOLD`` and terminal action ``TA-HOLD-NO-WRITE``; counter equality alone is not a substitute.

Limits the boot binding must enforce
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PROVISIONAL, pending coordinator approval. The complete canonical success-mark envelope is at most 4,096 bytes inclusive; the core and mark payloads are each at most 3,072 bytes; boot objects have depth at most 8, at most 32 properties, arrays at most 32 entries, and at most 32 checks. The complete core envelope maximum is not stated by the candidate and is BLK-13; the U-Boot container reserves 4,096 bytes for it and rejects a ratified bound above that as a design conflict requiring a new ruling rather than silently widening. ``Digest`` is ``sha256:`` plus 64 lowercase hexadecimal characters. ``SlotId`` is exactly ``slot-a``, ``slot-b``, or ``recovery``. ``BoardId`` is ``apple:`` plus a lowercase token of 1 to 64 bytes. ``Generation`` and counters are unsigned 64-bit values that never wrap or reset; the value 18,446,744,073,709,551,615 is a hard stop that returns ``BOOT_COUNTER_FAILURE`` before any increment.

Failure vocabulary used by the boot consumer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PROVISIONAL, pending coordinator approval. Every boot decision uses exactly one registered local code and one terminal action from `Local failure codes`_ and `Failure code/path/phase/result matrix`_. ``HOLD`` means the current record remains unchanged and no launch occurs. ``HALT`` means no write and no launch. ``REJECT`` means the input object is absent and cannot authorize a later phase. ``BUILD_FAIL`` means release admission stops before an image exists. Unknown fields, duplicate keys, and canonicalization failures are all classified by the single ``PARSE_SCHEMA_FAILURE`` row. There is no local remapping at the OS boundary; the exact registered code remains the diagnostic identity.

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

The exact linearizable ordering is defined in `Commit protocol and crash recovery`_. The floor reservation is durable before BCR bytes change, the floor commit is the single transaction linearization point, and both BCR committed markers are required before launch. If ``floor_value`` is ``UINT64_MAX`` or the next reservation would overflow u64, the exact result is ``(BOOT_COUNTER_FAILURE, atomic_record.counter, freshness, HOLD)`` with terminal action ``TA-HOLD-NO-WRITE`` before any floor or BCR write. A failed device identity check is ``BOOT_DEVICE_BINDING_FAILURE``; a failed floor or BCR transaction is ``BOOT_RECORD_COMMIT_FAILURE``; each uses its single matrix tuple and no write or launch follows it. A stale full-disk image or clone cannot lower the external floor or change the current-device identity.

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
     - Only the ratified ``committed`` value constructs the trusted record; ``prepared`` is a recovery marker and an unknown value is ``BOOT_RECORD_COMMIT_FAILURE``
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
     - Enforce one ``document_id`` to one ``payload_digest`` and permanent non-reuse; a second digest is ``DOCUMENT_ID_REUSE`` and two durable divergent digests are ``DOCUMENT_ID_FORK``
   * - ``Trusted<AtomicBootRecord>`` constructor
     - Generated F-02 constructor over the verified canonical journal record and ``Trusted<TrustContext>``
     - No local constructor or consumer-local alias
     - The only successful result is ``Trusted<AtomicBootRecord>``; unavailable or mismatched F-02/F-03 authority is ``BINDING_INTEGRITY_FAILURE`` with result ``HOLD``

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
     - Exact imported F-02 ``AtomicBootRecord`` phase marker: ``prepared`` or ``committed``; the wire values are supplied by the ratified F-02 lock, and an unknown value invalidates the copy
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
     - Mark/core/header authentication, signature, role, domain, context, or container-state check failed
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
     - ``BOOT_RECORD_COMMIT_FAILURE``
     - The BCR/floor transaction, repair, or writer serialization cannot complete the fixed durable protocol
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
     - ``BOOT_MARKER_COMMIT_FAILURE``
     - A BSM header, core, mark, or clear write did not complete and read back exactly
   * - 37
     - ``CI_CONFORMANCE_FAILURE``
     - A required generated fixture, validator, or CI gate is absent, fails, or is unobserved
   * - 38
     - ``QUALIFICATION_FAILURE``
     - Required board/profile/physical qualification evidence is absent or fails
   * - 39
     - ``RELEASE_PROFILE_FAILURE``
     - The resolved release configuration is absent, mismatched, or not fully closed
   * - 40
     - ``BOOT_ARTIFACT_ID_FAILURE``
     - An artifact ID cannot pass the strict byte grammar or bounded safe-join/file-object checks
   * - 41
     - ``BOOT_ARTIFACT_ROLE_FAILURE``
     - An artifact role is missing, duplicated, or has more than one manifest entry

The codes 14 through 41 are U-Boot-local diagnostic codes for conditions the F-02 vocabulary does not name at this boundary. They are never written into an F-02 payload and are never renamed at a consumer boundary. The code string printed, stored in BCR-18/BCR-S05, and carried in any diagnostic evidence is the exact registry string. Each registry entry has exactly one matrix row, one canonical path, one phase, one result, and one terminal action in `Failure code/path/phase/result matrix`_.

Failure code/path/phase/result matrix
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following is the closed mapping used by every transition, copy case, success-mark case, hostile fixture, residual, and blocked constant. The path is the first canonical input path or physical resolver; later checks cannot replace its result. Result values are singletons: ``PASS``, ``REJECT``, ``HOLD``, ``HALT``, or ``BUILD_FAIL``. Terminal actions are closed: ``TA-CONTINUE``, ``TA-REJECT-OBJECT``, ``TA-HOLD-NO-WRITE``, ``TA-HALT-NO-LAUNCH``, ``TA-BUILD-FAIL``, ``TA-CI-FAIL``, and ``TA-QUALIFICATION-FAIL``. Each token has one meaning, and a row contains one result and one action.

Simultaneous faults use one total precedence. U-Boot assigns every observed fault ``(phase_rank, path_rank)`` and selects the minimum pair. The phase ranks are fixed integers: 0 ``ratification``, 1 ``device-binding``, 2 ``physical-resolution``, 3 ``record-authentication``, 4 ``record-replay``, 5 ``state-validation``, 6 ``candidate-detection``, 7 ``manifest-parse``, 8 ``signature-context``, 9 ``manifest-projection``, 10 ``manifest-expiry``, 11 ``promotion-admission``, 12 ``artifact-admission``, 13 ``profile-admission``, 14 ``freshness``, 15 ``success-mark``, 16 ``transition``, 17 ``durable-commit``, 18 ``recovery-admission``, 19 ``release-profile``, 20 ``ci-gate``, and 21 ``qualification-gate``. ``none`` is the no-fault code-0 row and never competes with a fault. Within a phase, ``path_rank`` is the unsigned-byte lexicographic order of the unique canonical path in this matrix. The selected row is the only reported tuple; later faults never replace it, and no fault may emit a set of codes or outcomes.

The closure guard extracts the numeric registry set and the matrix set and requires both to equal ``{0, 1, ..., 41}``; it requires every non-zero registry string to occur in exactly one matrix row and every non-zero matrix path to occur in exactly one row. It also requires every fixture, transition, copy case, success-mark case, residual, and blocked constant to reference an existing matrix tuple. A planted unknown code, duplicate code, missing row, duplicate path, code-17 alias, multi-result token, or multi-action token rejects the design model. Code 17 is only ``BOOT_RECORD_COMMIT_FAILURE`` and no prose may report it under another name.

.. list-table::
   :header-rows: 1
   :widths: 6 28 18 12 36

   * - Code
     - First-failure path
     - Phase
     - Result
     - Terminal action
   * - 0 ``none``
     - ``none``
     - none
     - ``PASS``
     - ``TA-CONTINUE``
   * - 1 ``BOOT_FALLBACK_FAILURE``
     - ``transition.fallback_target``
     - transition
     - ``HALT``
     - ``TA-HALT-NO-LAUNCH``
   * - 2 ``BOOT_MARKER_AUTH_FAILURE``
     - ``success_mark.authentication``
     - success-mark
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 3 ``BOOT_CONTEXT_MISMATCH``
     - ``success_mark.binding_tuple``
     - success-mark
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 4 ``BOOT_COUNTER_FAILURE``
     - ``atomic_record.counter``
     - freshness
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 5 ``BOOT_REQUIRED_CHECK_FAILURE``
     - ``boot_health.checks``
     - profile-admission
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 6 ``TRUST_BOUNDARY_FAILURE``
     - ``trust.input``
     - ratification
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 7 ``CROSS_DOCUMENT_MISMATCH``
     - ``manifest.cross_document``
     - signature-context
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 8 ``BINDING_INTEGRITY_FAILURE``
     - ``contract.authority_lock``
     - ratification
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 9 ``MANIFEST_EXPIRY_FAILURE``
     - ``manifest.expires_at``
     - manifest-expiry
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 10 ``EXPIRY_OR_REPLAY_FAILURE``
     - ``success_mark.replay_id``
     - record-replay
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 11 ``SIGNATURE_CONTEXT_MISMATCH``
     - ``manifest.signature_context``
     - signature-context
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 12 ``RESOURCE_LIMIT``
     - ``envelope.resource_limit``
     - manifest-parse
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 13 ``PARSE_SCHEMA_FAILURE``
     - ``envelope.schema``
     - manifest-parse
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 14 ``OMARCHY_ARTIFACT_DIGEST_MISMATCH``
     - ``artifact.content_digest``
     - artifact-admission
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 15 ``OMARCHY_ARTIFACT_MISSING``
     - ``artifact.required_entry``
     - artifact-admission
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 16 ``OMARCHY_BCR_DIVERGENT``
     - ``bcr.copy_divergence``
     - record-authentication
     - ``HALT``
     - ``TA-HALT-NO-LAUNCH``
   * - 17 ``BOOT_RECORD_COMMIT_FAILURE``
     - ``bcr.commit.transaction``
     - durable-commit
     - ``HALT``
     - ``TA-HALT-NO-LAUNCH``
   * - 18 ``OMARCHY_BOARD_MISMATCH``
     - ``board_registry.match``
     - device-binding
     - ``HALT``
     - ``TA-HALT-NO-LAUNCH``
   * - 19 ``OMARCHY_LKG_INVALID``
     - ``slot.last_known_good``
     - state-validation
     - ``HALT``
     - ``TA-HALT-NO-LAUNCH``
   * - 20 ``BOOT_DEVICE_BINDING_FAILURE``
     - ``storage.device_installation_binding``
     - device-binding
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 21 ``BOOT_RECORD_DIGEST_FAILURE``
     - ``bcr.record_digest``
     - record-authentication
     - ``HALT``
     - ``TA-HALT-NO-LAUNCH``
   * - 22 ``BOOT_RECORD_SOURCE_FAILURE``
     - ``bcr.source_record``
     - record-authentication
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 23 ``BOOT_REPLAY_RESERVATION_FAILURE``
     - ``bcr.replay_reservation``
     - record-replay
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 24 ``BOOT_PROJECTION_FAILURE``
     - ``manifest.projection``
     - manifest-projection
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 25 ``BOOT_PROFILE_FAILURE``
     - ``boot_health.profile``
     - profile-admission
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 26 ``BOOT_FRESHNESS_FAILURE``
     - ``freshness.clock``
     - freshness
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 27 ``BOOT_PROMOTION_FAILURE``
     - ``manifest.promotion_receipt``
     - promotion-admission
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 28 ``BOOT_STORAGE_IDENTITY_FAILURE``
     - ``storage.esp.matches``
     - physical-resolution
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 29 ``BOOT_CANDIDATE_CONFLICT``
     - ``candidate_set``
     - candidate-detection
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 30 ``DOCUMENT_ID_REUSE``
     - ``atomic_record.permanent_tombstones.reuse``
     - state-validation
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 31 ``DOCUMENT_ID_FORK``
     - ``atomic_record.permanent_tombstones.fork``
     - state-validation
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 32 ``RELEASE_COMMAND_CLOSURE_FAILURE``
     - ``release_command_allowlist.enabled_command``
     - release-profile
     - ``BUILD_FAIL``
     - ``TA-BUILD-FAIL``
   * - 33 ``BOOT_PROVENANCE_FAILURE``
     - ``manifest.provenance_relation``
     - artifact-admission
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 34 ``BOOT_STATE_CONSISTENCY_FAILURE``
     - ``atomic_record.state``
     - state-validation
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 35 ``BOOT_RECOVERY_IDENTITY_FAILURE``
     - ``recovery.immutable_identity``
     - recovery-admission
     - ``HALT``
     - ``TA-HALT-NO-LAUNCH``
   * - 36 ``BOOT_MARKER_COMMIT_FAILURE``
     - ``bsm.commit.readback``
     - durable-commit
     - ``HOLD``
     - ``TA-HOLD-NO-WRITE``
   * - 37 ``CI_CONFORMANCE_FAILURE``
     - ``ci.required_fixture``
     - ci-gate
     - ``BUILD_FAIL``
     - ``TA-CI-FAIL``
   * - 38 ``QUALIFICATION_FAILURE``
     - ``qualification.board_profile``
     - qualification-gate
     - ``BUILD_FAIL``
     - ``TA-QUALIFICATION-FAIL``
   * - 39 ``RELEASE_PROFILE_FAILURE``
     - ``release.config``
     - release-profile
     - ``BUILD_FAIL``
     - ``TA-BUILD-FAIL``
   * - 40 ``BOOT_ARTIFACT_ID_FAILURE``
     - ``manifest.artifact_id``
     - artifact-admission
     - ``REJECT``
     - ``TA-REJECT-OBJECT``
   * - 41 ``BOOT_ARTIFACT_ROLE_FAILURE``
     - ``manifest.artifact_role``
     - artifact-admission
     - ``REJECT``
     - ``TA-REJECT-OBJECT``

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
     - Writes both copies with the exact ratified F-02 ``journal_schema``, ``commit_state``, authenticated source record, device/installation binding, floor receipt, replay reservation, and permanent-tombstone root; ``sequence = 1`` is evidence only, ``auth_algorithm = 1`` is provisioning-only, ``slot-a`` PENDING with ``attempts_used = 0``, ``attempt_counter = 0``, ``slot_generation = 1``, the staged manifest and complete profile digests, ``slot-b`` EMPTY, ``recovery`` PINNED with the immutable recovery identity, ``last_known_good_slot = 0``, and ``selected_slot = 1``. Any other content in an unkeyed record is invalid. If the canonical F-02/F-03 source, device binding, or floor cannot be authenticated, provisioning is not a release admission and the result is HOLD; there is no recovery-OS rewrite path.
   * - T-02
     - U-Boot
     - unkeyed to keyed
     - Folded into the first U-Boot commit protocol only after the ratification, device-binding, floor, and source-record gates: the record is rewritten with ``auth_algorithm = 2``, exact F-02 ``commit_state = committed``, a fresh MAC, ``record_digest``, and the reserved counter. In a non-release profile the record stays at algorithm 1 and cannot launch or produce a trusted BootContext.
   * - T-03
     - U-Boot
     - EMPTY, FAILED, or non-last-known-good ACCEPTED to PENDING (stage detection)
     - Exactly one eligible candidate exists. Its manifest passes the generated F-02 projection validator, the exact F-07 promotion receipt, board/channel/artifact/policy/anti-downgrade checks, complete BootCheck profile check, and document-lineage tombstone check before ``Trusted<PlatformManifest>`` is constructed. It names the current last-known-good manifest ID in ``components.boot_stack.rollback.previous_manifest_ids`` (or ``last_known_good_slot = 0``), every boot-path artifact verifies, and ``retry_limit`` is 1 through 8. If zero candidates exist, no staging occurs; if more than one exists, all are rejected with ``BOOT_CANDIDATE_CONFLICT`` and no PENDING write. Effect for exactly one: ``slot_generation + 1``, new ``lineage_id``, ``attempt_counter = 0``, ``attempts_used = 0``, ``attempt_limit``, complete profile digests, manifest/artifact digests, ``rollback_set_digest``, and ``selected_slot`` set to this slot. The prior document identity is appended to the permanent tombstone history (T-12), never discarded by descriptor overwrite.
   * - T-04
     - U-Boot
     - PENDING to PENDING (attempt consumption)
     - ``selected_slot`` is exactly the PENDING descriptor, ``attempts_used < attempt_limit``, and the independent authority durably records the next ``last_counter`` as ``reserved`` before any BCR byte changes. Effect: ``attempts_used + 1``, ``attempt_counter = reservation.counter``, ``last_counter = reservation.counter``, ``replay_reservation``, ``pending_time_unix``. The commit protocol writes copy 0 prepared, commits the external floor, writes copy 1 prepared, publishes both committed markers, and reads every boundary back before BootContext installation or launch. A maximum counter is rejected before this transition; it is never routed to T-06.
   * - T-05
     - U-Boot
     - PENDING to ACCEPTED
     - ``Trusted<BootHealthCore>`` was constructed only after the complete manifest-declared profile, source evidence, and freshness checks, and the separate success-mark validator accepts the exact ``attempt_counter``, core digest, descriptor, F-07 receipt, and rollback set. Effect: state ACCEPTED, ``last_known_good_slot`` set to this slot, BCR-S12, BCR-S13, BCR-S14 recorded, ``attempts_used`` retained as evidence, and the permanent lineage history extended. The former last-known-good slot remains ACCEPTED but is no longer last-known-good and becomes eligible for T-03. Committed before the mark container is cleared.
   * - T-06
     - U-Boot
     - PENDING to FAILED (budget exhausted)
     - ``attempts_used = attempt_limit`` and no valid success mark. Effect: state FAILED, ``slot_failure_code = 1`` (``BOOT_FALLBACK_FAILURE``), ``selected_slot`` set to the exact current LKG when non-zero, otherwise PINNED recovery. The FAILED descriptor keeps its identity and the permanent tombstone keeps its document ID and payload digest so the same candidate cannot be retried or forked. T-06 is budget exhaustion only; counter wrap is HF-15 and code 4 before any transition.
   * - T-07
     - U-Boot
     - PENDING to FAILED (explicit OS failure)
     - A ``Trusted<BootHealthCore>`` for exactly ``attempt_counter`` passed the complete profile validation and carries ``success = false`` and ``fallback.decision = recover`` with ``target_slot`` in the recomputed rollback set. Its ``fallback.failure_code`` must be exactly local code 1 (``BOOT_FALLBACK_FAILURE``); any other value is ``BOOT_STATE_CONSISTENCY_FAILURE`` before T-07. Effect as T-06 with ``slot_failure_code = 1``. A core with ``decision = hold`` leaves the slot PENDING and the next boot performs T-04; an invalid or stale core is not an explicit failure and cannot alter state.
   * - T-08
     - U-Boot
     - PENDING to FAILED (pre-launch validation failure)
     - The pending slot's manifest is missing, unsigned, expired, fails projection or F-07 promotion validation, not the recorded derived payload digest, or any boot-path artifact fails size or digest. Effect as T-06 with the first stable code from the failure matrix; a missing canonical F-02/F-03 dependency is HOLD and never a fallback.
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
     - A stale or invalid copy is rewritten from the selected valid record with its own ``copy_index``, tag, and CRC. ``sequence`` is unchanged. Performed after selection and before any transition commit; a repair read-back failure returns ``BOOT_RECORD_COMMIT_FAILURE`` at ``bcr.commit.transaction`` with terminal action ``TA-HALT-NO-LAUNCH``.
   * - T-12
     - U-Boot
     - descriptor overwrite (tombstone)
     - Part of T-03 and T-05: append the prior document ID, one and only one payload digest, generation, lineage, and manifest digest to the authenticated F-02 permanent-tombstone history before replacing a descriptor. An older mark or a document-ID fork can never match again; recycling a descriptor does not delete the history.

State validation is total and runs before selection or any write. ``selected_slot`` may be only a PENDING slot, the exact current LKG ACCEPTED descriptor, or the PINNED recovery descriptor. ``selected_slot`` may never name FAILED, EMPTY, an ACCEPTED descriptor that is not the LKG, an unknown value, or a contradictory descriptor. A PENDING or FAILED descriptor has ``attempt_limit`` in range and ``attempts_used`` within it; ``last_known_good_slot`` is zero or names exactly one ACCEPTED slot; at most one of ``slot-a`` and ``slot-b`` is PENDING; the recovery descriptor is EMPTY or PINNED; an ACCEPTED descriptor has non-zero BCR-S12, BCR-S13, and BCR-S14; and an unkeyed record has exactly the T-01 shape. Every invalid Cartesian combination returns ``BOOT_STATE_CONSISTENCY_FAILURE``, records the first canonical state path, performs no write or launch, and terminates in ``HOLD``. No assertion that a writer "derived" a field substitutes for validation.

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
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
   * - ``slot-a``
     - code 34; action=none; terminal=``HOLD``
     - code 0; action=T-04 then launch; terminal=``LAUNCH``
     - code 0; action=LKG verify; terminal=``LAUNCH``
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
   * - ``slot-b``
     - code 34; action=none; terminal=``HOLD``
     - code 0; action=T-04 then launch; terminal=``LAUNCH``
     - code 0; action=LKG verify; terminal=``LAUNCH``
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
   * - ``recovery``
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
     - code 0; action=immutable-recovery verify then launch; terminal=``LAUNCH``
   * - unknown
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``
     - code 34; action=none; terminal=``HOLD``

The matrix is exhaustive, not illustrative: each generated case records ``(selected_slot, descriptor states, last_known_good_slot, code, action, terminal_state)`` and the acceptance gate fails if the count is not exactly 2,500, if any tuple is missing or duplicated, or if any legal row lacks one deterministic launch or transition result.

Two-copy selection and recovery
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On every boot U-Boot first passes the ratification and device-binding gates, gathers all physical storage matches, and verifies the single same-disk ESP/BCR/BSM tuple before reading slot files. It then reads both copies and the imported floor-authority receipt. A copy is valid only if every check in `Record encoding`_, `atomic-boot-record-mapping`_, `Local failure codes`_, and the total state rules passes, including the live GPT identity. ``BCR-23`` is only the per-copy phase marker; the floor receipt and the canonical record form one transaction key ``(control_set_id, last_counter, replay_reservation_digest, record_digest)``. A transaction with a floor state of ``RESERVED`` is not launch-authoritative; a transaction with a floor state of ``COMMITTED`` is recoverable only when one valid prepared or committed copy carries the exact key. BCR-07 ``sequence`` and timestamps never participate in authority selection.

The external floor is an imported F-02/F-03 authority, not a local file. Its future-ratified ``floor-reservation/v1`` record has exactly ``floor_schema``, ``control_set_id``, ``installation_binding_digest``, ``floor_value``, ``counter``, ``replay_reservation_digest``, ``record_digest``, ``previous_floor``, ``state``, ``reservation_digest``, ``issued_at``, ``expires_at``, ``authority_binding_digest``, and ``burn_log_digest``. ``floor_value`` is the greatest committed counter; ``counter`` is the active candidate counter and is greater than ``floor_value`` only while ``RESERVED``. The only durable states are ``READY``, ``RESERVED``, ``COMMITTED``, and ``BURNED``. ``READY -> RESERVED`` and ``COMMITTED -> RESERVED`` allocate one counter greater than both the floor and every burned counter; ``RESERVED -> COMMITTED`` is one authenticated compare-and-swap and is the transaction linearization point; a failed or abandoned reservation performs ``RESERVED -> BURNED`` and retains its counter in the append-only burn log. ``BURNED -> RESERVED`` is permitted only with a new, greater counter. ``RESERVED`` never authorizes launch. ``COMMITTED`` authorizes only repair or publication of its exact transaction until both BCR copies carry that key and are read back; it never authorizes a second reservation while publication is incomplete. The floor authority's atomic receipt and its exact identity lock remain BLK-19; an unavailable or mismatched authority has the single ratification result in the failure matrix.

.. list-table::
   :header-rows: 1
   :widths: 8 34 58

   * - ID
     - Observation
     - Decision
   * - C2-01
     - Floor ``COMMITTED`` for the current key; both copies are committed with equal content and key
     - Use the current record; no repair and no new reservation
   * - C2-02
     - Both copies are committed with the same counter but different current transaction content
     - ``OMARCHY_BCR_DIVERGENT`` at ``bcr.copy_divergence``; no copy is selected and no write occurs
   * - C2-03
     - Floor ``COMMITTED`` for the current key; one current committed copy and one invalid or older-key copy
     - Use the current committed copy as the canonical source; repair the other copy with the exact current transaction and publish both markers. A repair read-back failure is ``BOOT_RECORD_COMMIT_FAILURE`` at ``bcr.commit.transaction`` and launches nothing
   * - C2-04
     - Floor ``COMMITTED`` for the current key; one current committed copy and one current prepared copy
     - Complete the prepared copy's commit marker, read it back, and select the resulting current record
   * - C2-05
     - Floor ``COMMITTED`` for the current key; one current prepared copy and one invalid or older-key copy
     - Use the prepared copy as the canonical source, write the other copy prepared, publish both markers in order, and select the resulting current record. A repair read-back failure is ``BOOT_RECORD_COMMIT_FAILURE`` at ``bcr.commit.transaction`` and launches nothing
   * - C2-06
     - Floor ``RESERVED`` for a current key; one old committed copy and one prepared copy carrying the reserved key
     - Burn the reservation, retain the old committed copy as authority, and repair the prepared copy from it; no transition is launched
   * - C2-07
     - Floor ``COMMITTED`` with zero current prepared or committed copies for its key
     - ``BOOT_RECORD_COMMIT_FAILURE`` at ``bcr.commit.transaction``; no write and no launch
   * - C2-08
     - Zero valid copies, floor ``READY`` or ``BURNED`` with no active transaction, and the immutable recovery identity verifies
     - Launch the PINNED recovery descriptor after its complete checks; no BCR write occurs
   * - C2-09
     - Zero valid copies, floor ``READY`` or ``BURNED`` with no active transaction, and the immutable recovery identity fails verification
     - ``BOOT_RECOVERY_IDENTITY_FAILURE`` at ``recovery.immutable_identity``; no write and no launch
   * - C2-10
     - GPT or exact partition identity cannot be resolved
     - ``BOOT_STORAGE_IDENTITY_FAILURE`` at ``storage.esp.matches``; no BCR or slot bytes are read
   * - C2-11
     - A copy's ``copy_index`` or partition GUID disagrees with its physical partition
     - ``BOOT_RECORD_SOURCE_FAILURE`` at ``bcr.source_record``; the copy is discarded and no record is selected
   * - C2-12
     - A keyed record and an unkeyed provisioning record are both readable
     - Select the keyed record only when its current floor key is ``COMMITTED`` and both-copy rules pass; the unkeyed record is rejected as non-release state
   * - C2-13
     - After reset, floor ``RESERVED`` and the old committed copy is intact
     - Perform the idempotent burn, repair the other copy from the old record, and retain the old record; no launch occurs
   * - C2-14
     - After reset, floor ``COMMITTED`` and one matching prepared copy remains
     - Resume the same transaction without a new reservation, finish both committed markers, and launch only after read-back

Commit protocol and crash recovery
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following protocol is executed for every transition in T-02 through T-09. It is one linearizable state machine, not a choice between copy order and floor order:

1. Re-resolve both partitions with the full identity tuple and read the authenticated floor receipt before any BCR write. A physical-resolution failure selects ``(BOOT_STORAGE_IDENTITY_FAILURE, storage.esp.matches, physical-resolution, HOLD)`` with ``TA-HOLD-NO-WRITE``; an unreadable or malformed floor transaction at this seam selects ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)`` with ``TA-HALT-NO-LAUNCH``. Either result writes nothing.
2. Reserve the next independent monotonic ``last_counter`` and replay reservation. The floor authority durably records ``state = RESERVED`` and the full transaction key before any BCR byte changes; this counter is burned on every failed transaction and is never reused. A reservation failure selects the durable-commit tuple above and launches nothing.
3. Build the complete canonical F-02 record in memory with ``commit_state = prepared``, the exact source record, device binding, ``record_digest``, floor key, profile projection, and permanent tombstone root. Write copy 0, flush, read back, and compare bytewise. Copy 1 remains the prior committed record at this point.
4. Atomically compare-and-swap the floor receipt from ``RESERVED`` to ``COMMITTED`` for this exact transaction key and verify the authenticated receipt. The floor's ``COMMITTED`` state is the sole linearization point; it makes the transaction the repair authority, but not launch authority.
5. Write copy 1 with the same prepared transaction, flush, read back, and compare. A mismatch selects the durable-commit tuple above because copy 0 and the committed floor carry the exact key.
6. Change copy 0's marker to ``committed``, write, flush, read back, and compare. Copy 1 remains prepared until this read-back succeeds; any failure selects the durable-commit tuple above.
7. Change copy 1's marker to ``committed``, write, flush, read back, and compare. Any failure selects the durable-commit tuple above. The transaction is launch-eligible only after this read-back and a bytewise equality check outside ``copy_index``, ``auth_tag``, and ``crc32c``.
8. Only after both committed copies and the committed floor are durable does U-Boot install BootContext and launch. Any reset before this point has the exact no-launch outcome in the power-cut table.

After every reset, repair reads the floor receipt first. A ``RESERVED`` receipt is atomically changed to ``BURNED`` with its counter retained in ``burn_log_digest``; the old committed record remains the only record authority and any prepared reservation-key copy is overwritten from that old record. A ``COMMITTED`` receipt requires one current-key prepared or committed copy; U-Boot uses that copy as the canonical source, reconstructs the other copy, and completes the fixed marker order without a new reservation. A ``READY`` or ``BURNED`` receipt permits only the current fully committed record or the PINNED recovery identity. A malformed receipt, an impossible state transition, or a ``COMMITTED`` receipt with no current-key copy returns ``BOOT_RECORD_COMMIT_FAILURE`` at ``bcr.commit.transaction`` and launches nothing. Repair never allocates a new counter, changes a manifest, selects a different slot, or treats copy order as authority. The recovery action is selected solely by the observed authenticated floor state and current transaction key, so it is idempotent across repeated resets.

Writer serialization: U-Boot runs single-threaded, and the writer keeps a static in-progress flag; a re-entrant call returns ``BOOT_RECORD_COMMIT_FAILURE`` at ``bcr.commit.transaction`` during ``durable-commit`` with result ``HALT`` and terminal action ``TA-HALT-NO-LAUNCH`` without writing. Only ``omarchy release`` reaches the writer. The diagnostic subcommands, the EFI runtime, and every other command have no path to it. The durable marker order is fixed: floor ``RESERVED``, copy 0 ``prepared``, floor ``COMMITTED``, copy 1 ``prepared``, copy 0 ``committed``, copy 1 ``committed``.

The sandbox injects exactly twelve named power cuts. A cut is made at the named durable boundary, and the next-boot authority is the single row below. ``NO_LAUNCH`` describes the interrupted boot; it is not a release result and cannot be replaced by a recovery guess. CP-04 cuts the atomic floor compare-and-swap before its read-back; the authenticated read-back is the sole observation used by the total repair dispatcher: ``RESERVED`` is burned and ``COMMITTED`` is resumed, while any third byte state is invalid and returns ``BOOT_RECORD_COMMIT_FAILURE`` at ``bcr.commit.transaction``. This is one deterministic phase rule, not an unresolved alternative expected result. T-05 uses the same CP-01 through CP-10 BCR protocol; it adds no thirteenth BCR case.

.. list-table::
   :header-rows: 1
   :widths: 9 24 23 30 14

   * - Cut
     - Injection boundary
     - Durable marker after reset
     - Next-boot authority and action
     - Interrupted boot
   * - CP-01
     - Before floor reservation
     - Previous committed floor and both previous committed copies
     - Previous committed record; retry the same transition with a fresh reservation
     - ``NO_LAUNCH``
   * - CP-02
     - After floor reservation, before copy 0 prepared write
     - Floor ``RESERVED``; both previous committed copies
     - Burn reservation; previous committed record remains authority
     - ``NO_LAUNCH``
   * - CP-03
     - During copy 0 prepared write
     - Floor ``RESERVED``; copy 0 invalid; copy 1 previous committed
     - Burn reservation; repair copy 0 from copy 1; previous committed record remains authority
     - ``NO_LAUNCH``
   * - CP-04
     - During atomic floor commit
     - Floor receipt read-back contains the one atomically durable state; copy 0 prepared; copy 1 previous committed
     - Apply the floor-state dispatcher: burn a ``RESERVED`` reservation and retain the old record, or resume a ``COMMITTED`` reservation using copy 0; an invalid third state is ``BOOT_RECORD_COMMIT_FAILURE`` at ``bcr.commit.transaction``
     - ``NO_LAUNCH``
   * - CP-05
     - After floor commit, before copy 1 prepared write
     - Floor ``COMMITTED``; copy 0 prepared; copy 1 previous committed
     - Resume the keyed transaction; write copy 1 prepared before marker publication
     - ``NO_LAUNCH``
   * - CP-06
     - During copy 1 prepared write
     - Floor ``COMMITTED``; copy 0 prepared; copy 1 invalid
     - Resume from copy 0 prepared; write copy 1 prepared and continue the same transaction
     - ``NO_LAUNCH``
   * - CP-07
     - After copy 1 prepared read-back, before copy 0 committed marker
     - Floor ``COMMITTED``; both copies prepared with the same key
     - Publish copy 0 committed, then copy 1 committed; no new reservation
     - ``NO_LAUNCH``
   * - CP-08
     - During copy 0 committed marker write
     - Floor ``COMMITTED``; copy 0 invalid; copy 1 prepared
     - Use copy 1 prepared as the transaction source; publish copy 0 committed, then copy 1 committed
     - ``NO_LAUNCH``
   * - CP-09
     - During copy 1 committed marker write
     - Floor ``COMMITTED``; copy 0 committed; copy 1 invalid
     - Use copy 0 committed as the transaction source; repair copy 1 and verify equality
     - ``NO_LAUNCH``
   * - CP-10
     - After both committed-copy read-backs, before BootContext installation
     - Floor ``COMMITTED``; both copies committed and equal
     - New committed record is authoritative; next boot proceeds from the committed descriptor
     - ``NO_LAUNCH``
   * - CP-11
     - During BSM core/mark/header publication
     - BCR committed; BSM header invalid
     - BSM is rejected; pending descriptor remains pending and next boot consumes its next budget attempt
     - ``NO_LAUNCH``
   * - CP-12
     - During BSM clear after T-05 BCR commit
     - BCR accepted; BSM header remains written
     - SM-17 clears the matching mark idempotently; accepted descriptor remains authoritative
     - ``NO_LAUNCH``

Every CP row captures both BCR copies, the floor receipt, the BSM header, the selected descriptor, and the terminal code. The cut-point model is design-only until the sandbox executes all twelve rows; absent execution is ``NOT EXECUTABLE``.

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
     - ``checks_digest`` differs from U-Boot's recomputation
     - ``BOOT_REQUIRED_CHECK_FAILURE``; hold; clear
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
     - ``marked_at`` outside the freshness policy
     - ``BOOT_FRESHNESS_FAILURE``; hold; clear. If BLK-06 is unavailable or mismatched, no counter-only substitute is allowed and the release path HALTs
   * - SM-22
     - ``Trusted<BootHealthCore>`` exists but the mark is absent
     - ``BOOT_MARKER_AUTH_FAILURE``; no T-05; hold; clear
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

0. Verify the exact ratified F-02/F-03 contract, generated bindings, authenticated opaque-predecessor interface envelope, device/installation binding, and independent monotonic-floor receipt. Missing or mismatched inputs produce ``HOLD`` before storage or slot access.
1. Resolve the board: read root ``compatible`` and ``model`` from the working device tree, match exactly one ``board-registry/v1`` board under ``identity_match.linux`` (delivery form BLK-11), and derive ``board_id``. Zero or more than one match is ``OMARCHY_BOARD_MISMATCH`` and HALT.
2. Resolve storage: gather all GPT disks and all partitions; resolve the Apple ESP by gathering every partition matching the exact ``asahi,efi-system-partition`` UUID and require exactly one match. Resolve BCR-0, BCR-1, and BSM by their complete typed identity tuples and require one live GPT disk GUID and parent/stable-ID relation for all of them. Zero or multiple ESP matches, duplicate UUIDs, or cross-disk tuples are ``BOOT_STORAGE_IDENTITY_FAILURE`` before any slot file is read. The current first-ESP fallback in ``asahi_esp_devpart()`` is excluded from the release profile (NI-13).
3. Read and select the BCR per `Two-copy selection and recovery`_; perform T-11 only when it is an authenticated same-disk repair and the independent floor permits it.
4. Consume and evaluate the BSM per `Mark evaluation cases`_; construct the trusted core first and the separate mark second; commit T-05 or T-07 only if both pass, then clear the container.
5. Stage detection: gather both ``slot-a`` and ``slot-b`` candidates that are not the exact current PENDING or LKG descriptors. Require exactly one F-07-authorized eligible candidate, including its promotion receipt, generation, lineage, rollback, board/profile, artifact, and anti-downgrade checks. Zero candidates means no staging; more than one is ``BOOT_CANDIDATE_CONFLICT`` with no PENDING write and ``HOLD``.
6. Select only the total state-machine outcomes: if ``selected_slot`` is exactly PENDING and ``attempts_used < attempt_limit``, reserve the next external counter and commit T-04; if its budget is exhausted, commit T-06 and re-select; if ``selected_slot`` is the exact LKG ACCEPTED descriptor, verify it and use T-09 on failure; if ``selected_slot`` is PINNED recovery, verify the immutable recovery identity and full recovery policy. FAILED, EMPTY, non-LKG ACCEPTED, unknown, and contradictory states are ``BOOT_STATE_CONSISTENCY_FAILURE`` and HALT.
7. Verify the slot: strict-parse and canonicalize the manifest, derive only ``sha256(JCS(payload))``, verify the signer and context, run the generated F-02 projection validator, verify the exact F-07 promotion receipt, then construct ``Trusted<PlatformManifest>``. Compare the derived digest to BCR-S08; check channel, board target, registry digest, consumer API, complete profile, anti-downgrade policy, and every declared artifact. On failure commit T-08 only when a valid transition is permitted; never replace a missing canonical dependency with recovery or a stable signature.
8. Install the BootContext into a working copy of the device tree per `BootContext transport`_ (release mode only; T-10 boots install the diagnostic marker instead).
9. Launch: call ``efi_binary_run()`` with the verified GRUB image bytes already in memory, the working device tree, and no initrd. GRUB then loads only the files named by the verified configuration.
10. If ``efi_binary_run()`` returns, the attempt is already consumed; print the return status, and reset. The next boot continues the bounded budget.

Steps 1 through 8 complete before any GRUB byte executes. No step reads an environment variable for authority; the command reads the control device tree through ``gd->fdt_blob`` and the block devices through the driver model.

GRUB artifact identity and configuration binding
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The slot layout on the ESP is fixed: ``EFI/OMARCHY/slots/A/``, ``EFI/OMARCHY/slots/B/``, and ``EFI/OMARCHY/recovery/``. Each directory contains ``manifest.json`` (the complete ``omarchy-signed/v1`` envelope of the slot manifest) and the artifact files named by that manifest. The file name of each artifact inside the slot is exactly its manifest ``artifact_id`` with the ``artifact:`` prefix removed; the manifest is the only mapping from role to file. The exact artifact IDs and media types for the GRUB image, the GRUB configuration, the U-Boot image, the kernel image, the initramfs, and the DTB set are BLK-09. Until ratified, the roles are referred to here by manifest ``kind`` and position, never by an invented file name. Every future BLK-09 value must satisfy `Artifact ID grammar and safe join`_.

Artifact ID grammar and safe join
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The future-ratified ``artifact_id`` grammar is strict and byte-oriented: ``artifact_id = "artifact:" name``; ``name = ALNUM_LOWER | ALNUM_LOWER ALNUM_MIDDLE{0,62} ALNUM_LOWER``; ``ALNUM_LOWER`` is one ASCII byte in ``a`` through ``z`` or ``0`` through ``9``; and ``ALNUM_MIDDLE`` is one byte in ``a`` through ``z``, ``0`` through ``9``, ``.``, ``-``, or ``_``. The prefix is exactly nine ASCII bytes, the name is 1 through 64 bytes, and the complete ID is 10 through 73 bytes. The first and last name bytes are always lowercase ASCII alphanumeric, so ``.``, ``..``, leading/trailing ``.``, ``-``, and ``_`` are rejected by the grammar. Uppercase ASCII, non-ASCII and Unicode bytes, control bytes, NUL, ``/``, and ``\`` are rejected before any string conversion. The grammar has one path component only, so directory depth is exactly zero after the fixed slot root. The exact BLK-09 role values are not supplied by this document; the future import must prove that every value fits this grammar.

Safe join is bounded and performed only after the manifest is trusted. U-Boot selects exactly one literal root from ``EFI/OMARCHY/slots/A``, ``EFI/OMARCHY/slots/B``, and ``EFI/OMARCHY/recovery``; strips exactly the nine-byte prefix; checks the byte length, every byte, first/last byte, separator count, and NUL absence in bounded memory; checks ``root_length + 1 + name_length <= 128`` with overflow-safe arithmetic; and constructs exactly ``root + "/" + name``. It performs no normalization, case folding, separator substitution, percent decoding, Unicode normalization, prefix matching, or fallback filename lookup. The filesystem operation is no-follow and requires one regular file whose containing directories are not symlinks or mount points; a symlink, directory, device node, mount point, or other non-regular object is rejected before file bytes are read. The declared size and content digest are then checked. A failed grammar or safe-join check is ``BOOT_ARTIFACT_ID_FAILURE`` at ``manifest.artifact_id`` during artifact-admission with result ``REJECT`` and terminal action ``TA-REJECT-OBJECT``. Duplicate or missing role cardinality is ``BOOT_ARTIFACT_ROLE_FAILURE`` at ``manifest.artifact_role``; source, recipe, or content provenance is ``BOOT_PROVENANCE_FAILURE`` at ``manifest.provenance_relation``. The exact injected slash, backslash, NUL, dot, dotdot, Unicode, uppercase, symlink, device, overlength, and depth cases are HF-65 through HF-75.

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

Validation by the Linux consumer (K-01 owned): read the three properties from ``/proc/device-tree/chosen``; require ``boot-mode = release``; recompute and compare the SHA-256 property; strictly parse the record under the bounded boot binding; resolve the same exact physical disk and typed parent/stable-ID relation; read both BCR copies read-only, select by the same precedence, verify the complete F-02 journal fields and independent floor, and require BCR-24/BCR-25/BCR-27/BCR-28/BCR-29 to equal the context fields; require the descriptor tuple for ``slot_id`` to equal every record field; construct ``Trusted<AtomicBootRecord>`` through the generated F-02 constructor and then ``Trusted<BootContext>`` through the only constructor. Any failure selects the single stable code from `Failure code/path/phase/result matrix`_; the health writer refuses to run, and no container write occurs.

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

HALT behavior is deterministic: print the local code string, both copies' census, the slot that was attempted, and the reason; then drop to the console with the banner ``OMARCHY RELEASE BOOT HALTED: manual actions are untrusted``. The release diagnostic allowlist is generated as ``release-command-allowlist/v1`` from the resolved release ``.config`` and the complete command-dispatch inventory. Its declared set is exactly ``omarchy``, ``usb``, ``reset``, and ``help``. ``usb`` is present because the immutable preboot string invokes ``usb start`` to initialize the USB HID keyboard; it is not a storage or boot selector. The inventory records every enabled command object, hidden command, alias, dynamic subcommand, and Kconfig-selected dispatch object; a command not in the four-item set is ``RELEASE_COMMAND_CLOSURE_FAILURE`` and fails the build gate. The current tree has no resolved release ``.config`` or generated inventory, so this is an unimplemented gate, not a four-command result.

The generated inventory explicitly covers inherited ``font`` and ``smbios`` commands; generic boot, bootflow, bootmeth, EFI boot manager, direct EFI, direct kernel, removable-media, NVMe, filesystem, serial-load, network, PXE, DHCP, media, environment, EFI-variable, script, hidden, alias, and dynamically registered commands. The USB category has exactly one permitted dispatch object, ``usb``, and its only release invocation is the immutable ``usb start`` preboot string. Every category other than the four allowlisted objects must be absent from the resolved release dispatch inventory. The allowlist validator fails for any enabled command outside the set, including one introduced through a Kconfig default or a command object not named in the static table; enumeration order is never authority.

The pinned source closes the manual keyboard path at the command seam: ``common/main.c`` executes the immutable ``preboot`` string when ``CONFIG_USE_PREBOOT=y``; ``cmd/Kconfig`` and ``cmd/Makefile`` compile and dispatch ``usb`` only when ``CONFIG_CMD_USB=y``; ``cmd/usb.c`` implements ``usb start`` by calling ``usb_init()``; and ``board/apple/mac/mac.env`` supplies ``stdin=serial,usbkbd,spikbid,mtpkbd``. The release profile therefore compiles ``CONFIG_CMD_USB=y``, keeps ``CONFIG_USB_KEYBOARD`` and the Apple keyboard options from the baseline, and sets ``CONFIG_USB_STORAGE=n``. After ``usb start`` initializes HID, the keyed autoboot stop string is the only automatic-to-manual boundary. A stopped console is untrusted and can invoke only the four allowlisted top-level command objects; no USB block device can become a boot source.

The resolved dispatch inventory is a set of top-level command objects, not a count or registration order. Each entry is the tuple ``(top_level_name, registration_source, selected_kconfig_symbols, hidden, aliases, dynamic_subcommands)``. The release closure guard expands every compiled command object, hidden command, alias, and dynamically registered object and requires the set of ``top_level_name`` values to equal exactly ``{omarchy, usb, reset, help}``; ``omarchy`` then has exactly the five subcommands in the command-surface table. A planted ``CONFIG_CMD_USB=n`` with ``preboot=usb start``, a mutable preboot value, an inherited ``font`` or ``smbios`` object, or any command outside that set is ``RELEASE_COMMAND_CLOSURE_FAILURE`` at ``release_command_allowlist.enabled_command`` during ``release-profile`` with result ``BUILD_FAIL`` and terminal action ``TA-BUILD-FAIL``. The guard records the resolved ``.config`` digest, inventory digest, and allowlist digest as one tuple; no inventory result is inferred from the defconfig text. Until those generated values exist, the seam remains NOT IMPLEMENTED.

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
   * - USB preboot
     - Exactly the ``usb`` command object and the immutable ``usb start`` invocation; no storage dispatch
     - ``CONFIG_CMD_USB=y``, ``CONFIG_USB_STORAGE=n``
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
     - ``y`` with immutable ``preboot=usb start``
     - ``cmd/Kconfig``, ``cmd/Makefile``, ``cmd/usb.c``, and ``common/main.c`` make the invocation executable; it initializes HID before keyed autoboot polling
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
   * - ``CONFIG_CMD_USB``
     - ``y``
     - Required by ``preboot=usb start``; the resulting ``usb`` dispatch object is the only allowlisted non-Omarchy diagnostic primitive
   * - ``CONFIG_CMD_MMC``, ``CONFIG_CMD_SCSI``, ``CONFIG_CMD_DFU``, ``CONFIG_CMD_FASTBOOT``, all network/PXE/DHCP command symbols
     - ``n``
     - Removes removable, media, provisioning, and network command paths; any symbol not named here is caught by the exhaustive allowlist
   * - ``CONFIG_FWU_MULTI_BANK_UPDATE``, ``CONFIG_BOOTCOUNT_LIMIT``
     - ``n``
     - The generic FWU trial counter and the environment bootcount are not the Omarchy state machine
   * - ``CONFIG_NVME_APPLE``, ``CONFIG_OF_UPSTREAM_BUILD_VENDOR``, ``CONFIG_USB``, ``CONFIG_USB_XHCI``, ``CONFIG_USB_DWC3``, ``CONFIG_USB_KEYBOARD``, and video options
     - as ``apple_m1_defconfig``
     - Hardware and USB HID baseline unchanged; USB mass storage remains explicitly disabled above
   * - ``CONFIG_SHA256``, ``CONFIG_CRC32C``
     - ``y``
     - Required by the BCR tag and CRC
   * - ``CONFIG_OMARCHY_BOOT``, ``CONFIG_CMD_OMARCHY``, ``CONFIG_OMARCHY_BOOT_RELEASE``, ``CONFIG_OMARCHY_BCR_AUTH_HMAC``, ``CONFIG_OMARCHY_BOOT_DIAG``
     - ``y``
     - Release state machine with untrusted diagnostics
   * - ``CONFIG_OMARCHY_RELEASE_COMMAND_ALLOWLIST``
     - ``y``
     - Requires the generated allowlist to equal the resolved dispatch inventory; any enabled command outside the diagnostic set fails the build/design gate

Enabled boot devices in release: the Apple NVMe namespace holding the ESP, resolved by UUID. Enabled boot methods: the ``omarchy release`` state machine. Recovery path: the PINNED recovery slot, verified against the same manifest rules. Console policy: reachable only through the keyed stop string, and every console action is labeled untrusted. Each failure selects exactly one row of the failure matrix and therefore one terminal action; no failure reaches a generic scan, removable medium, network source, alternate configuration file, or direct kernel image. No diagnostic action can write PENDING, ACCEPTED, FAILED, or a last-known-good change.

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
     - ``BOOT_PROVENANCE_FAILURE`` at ``predecessor.interface`` during entry; ``REJECT``
     - BLK-10 and F-03
   * - source tree/archive identity
     - F-04 builder
     - B-03/F-05 ``source.identity``
     - One ratified source kind only: immutable git tree preimage or immutable source archive preimage; no branch, tag, ref, or mutable checkout
     - source content digest, document digest, and payload digest
     - F-04 builder binding and signing context
     - per candidate; commit/archive identity and expiry
     - ``BOOT_PROVENANCE_FAILURE`` at ``manifest.components.boot_stack.source`` during source admission; ``REJECT``
     - BLK-18 and F-04
   * - artifact role
     - F-05 assembler
     - U-Boot/GRUB artifact consumer
     - ``artifact_id``, role, kind, media type, size, content bytes, manifest ID, and source identity
     - artifact content digest, manifest document digest, and derived payload digest
     - F-05 manifest-release context and artifact policy
     - manifest expiry and generation; anti-replay
     - ``BOOT_PROVENANCE_FAILURE`` at ``manifest.components.*.artifacts[]`` during artifact admission; ``REJECT``
     - BLK-09 and F-05
   * - recipe/toolchain lock
     - F-04 builder
     - B-03/F-05 ``build.inputs``
     - Canonical recipe, compiler/binutils/DT compiler/container identities, flags, and ordered patch queue
     - content, document, artifact, and payload digests
     - F-04 builder attestation and build context
     - per candidate and two-builder comparison
     - ``BOOT_PROVENANCE_FAILURE`` at ``provenance.recipe/toolchain`` during build admission; ``REJECT``
     - BLK-16 and F-04
   * - report lock
     - F-04/B-03
     - F-05/F-07 ``reports[]``
     - Canonical report bytes, report kind, tool version, input digests, and result
     - report content/document/payload digests
     - report signer and exact CI/build context
     - per candidate; report expiry and input-generation equality
     - ``BOOT_PROVENANCE_FAILURE`` at ``provenance.report_lock`` during promotion; ``REJECT``
     - F-04 and B-03
   * - provenance aggregate
     - F-04/F-05
     - U-Boot ``manifest.provenance`` and F-07 closure
     - Canonical ordered relation set linking source, recipe, toolchain, report, artifact, license, and builder observations
     - aggregate document/content/payload digest plus each relation digest
     - F-04/F-05 attestation and exact promotion context
     - candidate generation, expiry, and anti-replay
     - ``BOOT_PROVENANCE_FAILURE`` at ``manifest.provenance`` during artifact admission; ``REJECT``
     - BLK-16, BLK-18, F-04, and F-05
   * - device-installation binding
     - F-02/F-03 authority
     - U-Boot ``storage.device_installation_binding``
     - Exact ``JCS(I)`` binding preimage in `Freshness and installation binding`_
     - binding document/content/payload digest and floor receipt digest
     - F-03 device authority/context
     - monotonic floor, issued-at, expiry, replay
     - ``BOOT_DEVICE_BINDING_FAILURE`` at ``storage.parent_relation`` during device binding; ``HOLD``
     - BLK-19
   * - F-07 promotion
     - F-07 promotion terminal
     - U-Boot ``manifest.admission.promotion_receipt``
     - Canonical receipt without signature, including candidate, rollback, full closure, board/profile/qualification, legal/public-ledger, channel, manifest, generation, lineage, health, signer/context, and anti-replay fields
     - receipt document/content/payload digest and manifest derived payload digest
     - F-07 owner-authorized signer and promotion context
     - issued-at, expiry, generation, lineage, replay reservation
     - ``BOOT_PROMOTION_FAILURE`` at ``manifest.payload.promotion_receipt`` during promotion admission; ``REJECT``
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
     - ``BINDING_INTEGRITY_FAILURE`` at ``contract.authority_lock`` during ratification; no slot verifies
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
     - ``BOOT_PROVENANCE_FAILURE`` at ``manifest.provenance_relation``; ``REJECT``; ``TA-REJECT-OBJECT``
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

Each row is one reproducible fixture with one stable ``(code, path, phase, result)`` tuple, one terminal action, and one rejecting component. HF-01 through HF-51 and HF-63 through HF-75 contain one mutation against an accepted state. HF-17 and HF-52 through HF-62 are the twelve named power-cut fixtures. For a power-cut row, the tuple is the injected durable-commit observation and the final column is the interrupted run's mandatory ``NO_LAUNCH`` state; the next-boot authority is the single repair decision in the corresponding CP-01 through CP-12 row. HF-76 through HF-79 are explicit simultaneous-fault fixtures; their precedence is defined by the total matrix order. No fixture has an expected code set, result set, action set, delegated case, or alternative expected outcome. These fixtures are required test content; none has run (NI-05).

.. list-table::
   :header-rows: 1
   :widths: 7 25 38 30

   * - ID
     - Fixture
     - Mutation
     - Expected tuple; action; terminal
   * - HF-01
     - ``envelope-replay``
     - Present one previously accepted mark envelope byte-for-byte on a later boot
     - ``(BOOT_COUNTER_FAILURE, atomic_record.counter, freshness, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-02
     - ``envelope-transplant``
     - Move one valid mark container from another board's control set into this BSM
     - ``(BOOT_DEVICE_BINDING_FAILURE, storage.device_installation_binding, device-binding, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-03
     - ``typed-digest-substitution``
     - Replace the mark's ``manifest_digest`` with one document-ID digest
     - ``(BOOT_CONTEXT_MISMATCH, success_mark.binding_tuple, success-mark, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-04
     - ``wrong-board``
     - Set the mark's ``board_id`` to one sibling board
     - ``(BOOT_CONTEXT_MISMATCH, success_mark.binding_tuple, success-mark, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-05
     - ``wrong-manifest``
     - Bind the mark to the current last-known-good manifest
     - ``(BOOT_CONTEXT_MISMATCH, success_mark.binding_tuple, success-mark, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-06
     - ``wrong-slot``
     - Set ``slot_id = slot-a`` while ``slot-b`` is pending
     - ``(BOOT_CONTEXT_MISMATCH, success_mark.binding_tuple, success-mark, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-07
     - ``wrong-generation``
     - Set ``slot_generation`` one below the descriptor
     - ``(BOOT_CONTEXT_MISMATCH, success_mark.binding_tuple, success-mark, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-08
     - ``wrong-lineage``
     - Set ``lineage_id`` to one tombstoned lineage
     - ``(BOOT_CONTEXT_MISMATCH, success_mark.binding_tuple, success-mark, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-09
     - ``wrong-source-generation``
     - Reset ``source_generation`` to 0 in the mark
     - ``(BOOT_CONTEXT_MISMATCH, success_mark.binding_tuple, success-mark, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-10
     - ``forged-mark-signature``
     - Flip one mark signature byte
     - ``(BOOT_MARKER_AUTH_FAILURE, success_mark.authentication, success-mark, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-11
     - ``forged-mark-role``
     - Set ``signer_role = manifest-release`` on one mark
     - ``(BOOT_MARKER_AUTH_FAILURE, success_mark.authentication, success-mark, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-12
     - ``missing-mark``
     - Set the BSM container state to CONSUMED after one pending boot
     - ``(BOOT_MARKER_AUTH_FAILURE, success_mark.authentication, success-mark, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-13
     - ``replayed-mark-generation``
     - Set ``marker_generation`` equal to BCR-S12
     - ``(EXPIRY_OR_REPLAY_FAILURE, success_mark.replay_id, record-replay, REJECT)``; ``TA-REJECT-OBJECT``; no launch
   * - HF-14
     - ``counter-reset``
     - Set one copy's ``attempt_counter`` below its transaction counter
     - ``(BOOT_COUNTER_FAILURE, atomic_record.counter, freshness, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-15
     - ``counter-wrap``
     - Set ``last_counter = UINT64_MAX`` before T-04 reservation
     - ``(BOOT_COUNTER_FAILURE, atomic_record.counter, freshness, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-16
     - ``both-copies-corrupt``
     - Replace both BCR blocks with random bytes
     - ``(BOOT_RECORD_DIGEST_FAILURE, bcr.record_digest, record-authentication, HALT)``; ``TA-HALT-NO-LAUNCH``; no launch
   * - HF-17
     - ``power-cut-cp01``
     - Cut power before floor reservation
     - ``(0, none, none, PASS)``; ``TA-CONTINUE``; no launch during cut
   * - HF-18
     - ``stale-copy-selection``
     - Mark copy 0 prepared with a larger sequence while copy 1 is the previous committed record
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; no launch
   * - HF-19
     - ``concurrent-writer``
     - Re-enter the writer while its in-progress flag is set
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; no launch
   * - HF-20
     - ``power-loss-before-flush``
     - Cut power during the copy 0 prepared write
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; no launch
   * - HF-21
     - ``power-loss-after-flush``
     - Cut power after both committed-copy read-backs
     - ``(0, none, none, PASS)``; ``TA-CONTINUE``; no launch during cut
   * - HF-22
     - ``malicious-removable-media``
     - Attach one USB disk containing ``/EFI/BOOT/bootaa64.efi``
     - ``(0, none, none, PASS)``; ``TA-CONTINUE``; no USB boot
   * - HF-23
     - ``network-media``
     - Expose one network boot server on the link
     - ``(0, none, none, PASS)``; ``TA-CONTINUE``; no network boot
   * - HF-24
     - ``mutable-environment``
     - Add one ``/ubootefi.var`` file to the ESP
     - ``(0, none, none, PASS)``; ``TA-CONTINUE``; no environment read
   * - HF-25
     - ``grub-context-stripping``
     - Remove the three context properties before the Linux handoff
     - ``(TRUST_BOUNDARY_FAILURE, trust.input, ratification, HOLD)``; ``TA-HOLD-NO-WRITE``; no mark
   * - HF-26
     - ``grub-context-mutation``
     - Change one byte of ``omarchy,boot-context`` before Linux
     - ``(TRUST_BOUNDARY_FAILURE, trust.input, ratification, HOLD)``; ``TA-HOLD-NO-WRITE``; no mark
   * - HF-27
     - ``direct-kernel-bypass``
     - Enable one direct-kernel command in the resolved release dispatch
     - ``(RELEASE_COMMAND_CLOSURE_FAILURE, release_command_allowlist.enabled_command, release-profile, BUILD_FAIL)``; ``TA-BUILD-FAIL``; no release image
   * - HF-28
     - ``rollback-recursion``
     - Stage one identical manifest after T-06 in the FAILED slot
     - ``(DOCUMENT_ID_REUSE, atomic_record.permanent_tombstones.reuse, state-validation, REJECT)``; ``TA-REJECT-OBJECT``; no staging
   * - HF-29
     - ``missing-last-known-good``
     - Exhaust a pending slot with ``last_known_good_slot = 0`` and a valid pinned recovery descriptor
     - ``(0, none, none, PASS)``; ``TA-CONTINUE``; launch pinned recovery
   * - HF-30
     - ``alternate-stable-writer``
     - Present one stable manifest without a valid manifest-release binding
     - ``(BOOT_PROMOTION_FAILURE, manifest.promotion_receipt, promotion-admission, REJECT)``; ``TA-REJECT-OBJECT``; no PENDING write
   * - HF-31
     - ``unkeyed-accepted-record``
     - Set ACCEPTED in an auth-algorithm-1 record
     - ``(BOOT_RECORD_SOURCE_FAILURE, bcr.source_record, record-authentication, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-32
     - ``lkg-overwritten-in-place``
     - Replace the LKG manifest with one different trusted manifest
     - ``(OMARCHY_LKG_INVALID, slot.last_known_good, state-validation, HALT)``; ``TA-HALT-NO-LAUNCH``; no slot retry
   * - HF-33
     - ``recovery-pin-mismatch``
     - Replace the pinned recovery manifest identity
     - ``(BOOT_RECOVERY_IDENTITY_FAILURE, recovery.immutable_identity, recovery-admission, HALT)``; ``TA-HALT-NO-LAUNCH``; no recovery launch
   * - HF-34
     - ``full-disk-rollback``
     - Restore an older disk snapshot after a newer counter reservation
     - ``(BOOT_COUNTER_FAILURE, atomic_record.counter, freshness, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-35
     - ``full-disk-clone``
     - Clone the complete control tuple to another device
     - ``(BOOT_DEVICE_BINDING_FAILURE, storage.device_installation_binding, device-binding, HOLD)``; ``TA-HOLD-NO-WRITE``; no slot read
   * - HF-36
     - ``manifest-projection-mismatch``
     - Change one top-level projection while retaining the signed component projection
     - ``(BOOT_PROJECTION_FAILURE, manifest.projection, manifest-projection, REJECT)``; ``TA-REJECT-OBJECT``; no trusted manifest
   * - HF-37
     - ``derived-digest-wire-injection``
     - Add one ``payload_digest`` property to the envelope
     - ``(PARSE_SCHEMA_FAILURE, envelope.schema, manifest-parse, REJECT)``; ``TA-REJECT-OBJECT``; no derived value read
   * - HF-38
     - ``derived-digest-mismatch``
     - Retain an old BCR-S08 after changing the canonical manifest payload
     - ``(CROSS_DOCUMENT_MISMATCH, manifest.cross_document, signature-context, REJECT)``; ``TA-REJECT-OBJECT``; no launch
   * - HF-39
     - ``invalid-check-class``
     - Set one signed check class outside the allowed profile set
     - ``(BOOT_PROFILE_FAILURE, boot_health.profile, profile-admission, REJECT)``; ``TA-REJECT-OBJECT``; no trusted core
   * - HF-40
     - ``invalid-measurement``
     - Set one measurement unit outside the declared profile
     - ``(BOOT_PROFILE_FAILURE, boot_health.profile, profile-admission, REJECT)``; ``TA-REJECT-OBJECT``; no mark acceptance
   * - HF-41
     - ``profile-limit-or-source-mismatch``
     - Change one ``failure_limit`` while retaining passing statuses
     - ``(BOOT_PROFILE_FAILURE, boot_health.profile, profile-admission, REJECT)``; ``TA-REJECT-OBJECT``; no trusted core
   * - HF-42
     - ``bcr-loss``
     - Erase both BCR copies while retaining a committed floor receipt
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; no write
   * - HF-43
     - ``missing-f07-receipt``
     - Remove the promotion receipt from one stable manifest
     - ``(BOOT_PROMOTION_FAILURE, manifest.promotion_receipt, promotion-admission, REJECT)``; ``TA-REJECT-OBJECT``; no PENDING write
   * - HF-44
     - ``inherited-command``
     - Leave one inherited ``font`` dispatch object enabled
     - ``(RELEASE_COMMAND_CLOSURE_FAILURE, release_command_allowlist.enabled_command, release-profile, BUILD_FAIL)``; ``TA-BUILD-FAIL``; no release image
   * - HF-45
     - ``selected-state-cartesian``
     - Set ``selected_slot = slot-a`` while its descriptor state is EMPTY
     - ``(BOOT_STATE_CONSISTENCY_FAILURE, atomic_record.state, state-validation, HOLD)``; ``TA-HOLD-NO-WRITE``; no write
   * - HF-46
     - ``divergent-recovery``
     - Change the recovery identity in one equal-counter committed copy
     - ``(OMARCHY_BCR_DIVERGENT, bcr.copy_divergence, record-authentication, HALT)``; ``TA-HALT-NO-LAUNCH``; no record selection
   * - HF-47
     - ``cross-disk-tuple``
     - Place the ESP on one disk and BCR records on another disk
     - ``(BOOT_STORAGE_IDENTITY_FAILURE, storage.esp.matches, physical-resolution, HOLD)``; ``TA-HOLD-NO-WRITE``; no slot read
   * - HF-48
     - ``duplicate-esp-uuid``
     - Give two partitions the exact ESP UUID
     - ``(BOOT_STORAGE_IDENTITY_FAILURE, storage.esp.matches, physical-resolution, HOLD)``; ``TA-HOLD-NO-WRITE``; no enumeration authority
   * - HF-49
     - ``dual-candidate``
     - Place two distinct F-07-authorized candidates in the non-current slots
     - ``(BOOT_CANDIDATE_CONFLICT, candidate_set, candidate-detection, HOLD)``; ``TA-HOLD-NO-WRITE``; no PENDING write
   * - HF-50
     - ``document-id-reuse``
     - Stage one prior document ID with a different payload digest
     - ``(DOCUMENT_ID_REUSE, atomic_record.permanent_tombstones.reuse, state-validation, REJECT)``; ``TA-REJECT-OBJECT``; no staging
   * - HF-51
     - ``document-id-fork``
     - Present two durable payload digests for one document ID
     - ``(DOCUMENT_ID_FORK, atomic_record.permanent_tombstones.fork, state-validation, REJECT)``; ``TA-REJECT-OBJECT``; no staging
   * - HF-52
     - ``power-cut-cp02``
     - Cut power after floor reservation before copy 0 prepared write
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; previous record authority
   * - HF-53
     - ``power-cut-cp03``
     - Cut power during copy 0 prepared write
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; previous record authority
   * - HF-54
     - ``power-cut-cp04``
     - Cut power during atomic floor commit
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; previous record authority
   * - HF-55
     - ``power-cut-cp05``
     - Cut power after floor commit before copy 1 prepared write
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; resume keyed transaction
   * - HF-56
     - ``power-cut-cp06``
     - Cut power during copy 1 prepared write
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; resume keyed transaction
   * - HF-57
     - ``power-cut-cp07``
     - Cut power after copy 1 prepared read-back before copy 0 committed marker
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; resume keyed transaction
   * - HF-58
     - ``power-cut-cp08``
     - Cut power during copy 0 committed marker write
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; resume keyed transaction
   * - HF-59
     - ``power-cut-cp09``
     - Cut power during copy 1 committed marker write
     - ``(BOOT_RECORD_COMMIT_FAILURE, bcr.commit.transaction, durable-commit, HALT)``; ``TA-HALT-NO-LAUNCH``; repair from copy 0
   * - HF-60
     - ``power-cut-cp10``
     - Cut power after both committed-copy read-backs before BootContext installation
     - ``(0, none, none, PASS)``; ``TA-CONTINUE``; no launch during cut
   * - HF-61
     - ``power-cut-cp11``
     - Cut power during BSM core/mark/header publication
     - ``(BOOT_MARKER_COMMIT_FAILURE, bsm.commit.readback, durable-commit, HOLD)``; ``TA-HOLD-NO-WRITE``; no mark
   * - HF-62
     - ``power-cut-cp12``
     - Cut power during BSM clear after T-05 BCR commit
     - ``(BOOT_MARKER_COMMIT_FAILURE, bsm.commit.readback, durable-commit, HOLD)``; ``TA-HOLD-NO-WRITE``; accepted record remains
   * - HF-63
     - ``unknown-bsm-enum``
     - Set ``container_state = 3`` in the BSM header
     - ``(BOOT_MARKER_AUTH_FAILURE, success_mark.authentication, success-mark, HOLD)``; ``TA-HOLD-NO-WRITE``; no mark
   * - HF-64
     - ``duplicate-artifact-role``
     - Add a second artifact with the same manifest role
     - ``(BOOT_ARTIFACT_ROLE_FAILURE, manifest.artifact_role, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no trusted artifact
   * - HF-65
     - ``artifact-slash``
     - Set one ID to ``artifact:boot/kernel``
     - ``(BOOT_ARTIFACT_ID_FAILURE, manifest.artifact_id, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no filesystem access
   * - HF-66
     - ``artifact-backslash``
     - Set one ID to ``artifact:boot\\kernel``
     - ``(BOOT_ARTIFACT_ID_FAILURE, manifest.artifact_id, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no filesystem access
   * - HF-67
     - ``artifact-nul``
     - Insert one NUL byte into an ID
     - ``(BOOT_ARTIFACT_ID_FAILURE, manifest.artifact_id, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no filesystem access
   * - HF-68
     - ``artifact-dot``
     - Set the name component to ``.``
     - ``(BOOT_ARTIFACT_ID_FAILURE, manifest.artifact_id, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no filesystem access
   * - HF-69
     - ``artifact-dotdot``
     - Set the name component to ``..``
     - ``(BOOT_ARTIFACT_ID_FAILURE, manifest.artifact_id, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no filesystem access
   * - HF-70
     - ``artifact-unicode``
     - Insert one non-ASCII Unicode byte sequence into an ID
     - ``(BOOT_ARTIFACT_ID_FAILURE, manifest.artifact_id, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no filesystem access
   * - HF-71
     - ``artifact-uppercase``
     - Replace one lowercase ASCII byte with uppercase ASCII
     - ``(BOOT_ARTIFACT_ID_FAILURE, manifest.artifact_id, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no filesystem access
   * - HF-72
     - ``artifact-symlink``
     - Make the resolved artifact directory entry a symlink
     - ``(BOOT_ARTIFACT_ID_FAILURE, manifest.artifact_id, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no file read
   * - HF-73
     - ``artifact-device``
     - Make the resolved artifact directory entry a device node
     - ``(BOOT_ARTIFACT_ID_FAILURE, manifest.artifact_id, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no file read
   * - HF-74
     - ``artifact-overlength``
     - Set the complete ID length to 74 bytes
     - ``(BOOT_ARTIFACT_ID_FAILURE, manifest.artifact_id, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no filesystem access
   * - HF-75
     - ``artifact-depth``
     - Set the name to contain one directory separator after the prefix
     - ``(BOOT_ARTIFACT_ID_FAILURE, manifest.artifact_id, artifact-admission, REJECT)``; ``TA-REJECT-OBJECT``; no filesystem access
   * - HF-76
     - ``simultaneous-storage-record``
     - Combine a wrong board identity with one corrupt BCR copy
     - ``(OMARCHY_BOARD_MISMATCH, board_registry.match, device-binding, HALT)``; ``TA-HALT-NO-LAUNCH``; no storage read
   * - HF-77
     - ``simultaneous-counter-mark``
     - Combine ``last_counter = UINT64_MAX`` with one stale success mark
     - ``(BOOT_COUNTER_FAILURE, atomic_record.counter, freshness, HOLD)``; ``TA-HOLD-NO-WRITE``; no launch
   * - HF-78
     - ``simultaneous-bcr-artifact``
     - Combine one torn BCR copy with one wrong artifact digest
     - ``(BOOT_RECORD_DIGEST_FAILURE, bcr.record_digest, record-authentication, HALT)``; ``TA-HALT-NO-LAUNCH``; no artifact read
   * - HF-79
     - ``simultaneous-promotion-artifact``
     - Combine one missing F-07 receipt with one invalid artifact ID
     - ``(BOOT_PROMOTION_FAILURE, manifest.promotion_receipt, promotion-admission, REJECT)``; ``TA-REJECT-OBJECT``; no staging

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
     - ``BINDING_INTEGRITY_FAILURE``
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
     - No hostile fixture in `Hostile fixtures`_ (HF-01 through HF-79) has been executed
     - B-04 hostile-fixture gate
     - HF-01 through HF-79 fixture inventory
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
     - ``BINDING_INTEGRITY_FAILURE``
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

Each constant is owned outside this lane. Until ratified, U-Boot has no value for it, cannot be built for release, and B-03/B-04 admission is BLOCKED. No value is guessed here. The rows below are also the BLK dependency/consumer rows: each BLK ID appears exactly once with its consumer, consumer API/path, rejection code, handoff artifact/digest, verification command, owner, and due-before gate. The rejection code in a BLK row applies only after the owner-supplied value has passed the authority lock; a missing or mismatched owner record, schema record, lock field, or generated binding always uses ``BINDING_INTEGRITY_FAILURE`` at ``contract.authority_lock`` during ratification. Missing or duplicate mapping blocks release and F-07 promotion.

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
     - ``OMARCHY_BCR_AUTH_KEY`` source, provisioning, and rotation; the key must be unavailable to the Linux OS at runtime. No absence of a key source permits algorithm 1 in release; it is a HOLD threat-model decision
     - BCR verifier/writer
     - ``bcr.auth_tag`` and F-03 key binding
     - ``BINDING_INTEGRITY_FAILURE``
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
     - ``BINDING_INTEGRITY_FAILURE``
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
     - ``DOCUMENT_ID_REUSE``
     - canonical document-lineage digest
     - document-ID grammar and fork fixtures
     - F-02
     - B-04 implementation admission
   * - BLK-08
     - ``lineage_id`` allocation rule and its exact authenticated source; no local deterministic derivation is accepted until F-02 ratifies the rule
     - lineage validator
     - ``atomic_record.lineage_id`` and tombstone history
     - ``DOCUMENT_ID_FORK``
     - lineage document/content/payload digest
     - lineage allocation and reuse fixtures
     - F-02
     - B-04 implementation admission
   * - BLK-09
     - Artifact IDs and media types for the U-Boot image, GRUB image, GRUB configuration, kernel image, initramfs, DTB set, and recovery payload; every imported ID must satisfy the byte grammar and every role must occur once
     - manifest/artifact consumer
     - ``manifest.components.*.artifacts[]``
     - ``BOOT_ARTIFACT_ID_FAILURE``
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
     - ``BOOT_DEVICE_BINDING_FAILURE``
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

The B-04 implementation gate is: all 14 C2 rows, all 23 SM rows, all 12 T rows, and all 79 HF rows have deterministic sandbox results; E-01 passes; the release ``.config`` resolves on a pinned builder with the two-builder comparison and exhaustive command inventory; every NI/BLK row has exactly one consumer mapping; and all 20 BLK constants are ratified. The physical gate additionally requires the Q-04 rows on disposable qualified hardware with rehearsed outer recovery. A green sandbox run, a U-Boot prompt, a recognized SoC, or a booting desktop is never qualification evidence.

Closing statement
-----------------

This document is design only. It contains no marker and no unresolved local design question: every external authority still open is a named BLOCKED constant with an owner and a due-before gate, and every implementation or evidence gap is a NOT IMPLEMENTED residual with a per-item consumer contract. F-02 and F-03 remain unratified for this consumer, B-03 and B-04 remain open and not started, their implementation admission remains BLOCKED, no build succeeded, no fixture or experiment ran, no hardware booted, and nothing here is DONE.
