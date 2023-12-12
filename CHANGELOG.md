# Changelog
All notable changes to this library will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this library adheres to Rust's notion of
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]
The entries below are relative to the `zcash_primitives::sapling` module as of
`zcash_primitives 0.13.0`.

### Added
- `sapling_crypto::BatchValidator` (moved from `zcash_proofs::sapling`).
- `sapling_crypto::SaplingVerificationContext` (moved from
  `zcash_proofs::sapling`).
- `sapling_crypto::builder` (moved from
  `zcash_primitives::transaction::components::sapling::builder`). Additional
  additions to this module:
  - `UnauthorizedBundle`
  - `InProgress`
  - `{InProgressProofs, Unproven, Proven}`
  - `{InProgressSignatures, Unsigned, PartiallyAuthorized}`
  - `{MaybeSigned, SigningParts}`
  - `SpendDescriptionInfo::value`
  - `SaplingOutputInfo`
  - `ProverProgress`
- `sapling_crypto::bundle` module:
  - The following types moved from
    `zcash_primitives::transaction::components::sapling`:
    - `Bundle`
    - `SpendDescription, SpendDescriptionV5`
    - `OutputDescription, OutputDescriptionV5`
    - `Authorization, Authorized, MapAuth`
    - `GrothProofBytes`
  - `Bundle::<InProgress<Unproven, _>>::create_proofs`
  - `Bundle::<InProgress<_, Unsigned>>::prepare`
  - `Bundle::<InProgress<_, PartiallyAuthorized>>::{sign, append_signatures}`
  - `Bundle::<InProgress<Proven, PartiallyAuthorized>>::finalize`
  - `Bundle::<InProgress<Proven, Unsigned>>::apply_signatures`
  - `Bundle::try_map_authorization`
  - `TryMapAuth`
  - `impl {MapAuth, TryMapAuth} for (FnMut, FnMut, FnMut, FnMut)`
    helpers to enable calling `Bundle::{map_authorization, try_map_authorization}`
    with a set of closures.
  - `testing` module, containing the following functions moved from
    `zcash_primitives::transaction::components::sapling::testing`:
    - `arb_output_description`
    - `arb_bundle`
- `sapling_crypto::circuit` module (moved from `zcash_proofs::circuit::sapling`).
  Additional additions to this module:
  - `{SpendParameters, OutputParameters}`
  - `{SpendVerifyingKey, PreparedSpendVerifyingKey}`
  - `{OutputVerifyingKey, PreparedOutputVerifyingKey}`
- `sapling_crypto::constants` module.
- `sapling_crypto::keys`:
  - `SpendAuthorizingKey`
  - `SpendValidatingKey`
- `sapling_crypto::note_encryption`:
  - `CompactOutputDescription` (moved from
    `zcash_primitives::transaction::components::sapling`).
  - `SaplingDomain::new`
  - `Zip212Enforcement`
- `sapling_crypto::prover::{SpendProver, OutputProver}`
- `sapling_crypto::tree::Node::{from_bytes, to_bytes}`
- `sapling_crypto::value`:
  - `ValueCommitTrapdoor::from_bytes`
  - `impl Sub<TrapdoorSum> for TrapdoorSum`
  - `impl Sub<CommitmentSum> for CommitmentSum`
- `sapling_crypto::zip32` module (moved from `zcash_primitives::zip32::sapling`).
- `impl Debug for sapling_crypto::keys::{ExpandedSpendingKey, ProofGenerationKey}`
- Test helpers, behind the `test-dependencies` feature flag:
  - `sapling_crypto::prover::mock::{MockSpendProver, MockOutputProver}`

### Changed
- `sapling_crypto`:
  - `BatchValidator::validate` now takes the `SpendVerifyingKey` and
    `OutputVerifyingKey` newtypes.
  - `SaplingVerificationContext::new` now always creates a context with ZIP 216
    rules enforced, and no longer has a boolean for configuring this.
  - `SaplingVerificationContext::{check_spend, final_check}` now use the
    `redjubjub` crate types for `rk`, `spend_auth_sig`, and `binding_sig`.
  - `SaplingVerificationContext::{check_spend, check_output}` now take
    the `PreparedSpendVerifyingKey` and `PreparedOutputVerifyingKey`
    newtypes.
  - `SaplingVerificationContext::final_check` now takes its `value_balance`
    argument as `V: Into<i64>` instead of
    `zcash_primitives::transaction::components::Amount`.
- `sapling_crypto::address::PaymentAddress::create_note` now takes its `value`
  argument as a `NoteValue` instead of as a bare `u64`.
- `sapling_crypto::builder`:
  - `SaplingBuilder` no longer has a `P: zcash_primitives::consensus::Parameters`
    type parameter.
  - `SaplingBuilder::new` now takes a `Zip212Enforcement` argument instead of a
    `P: zcash_primitives::consensus::Parameters` argument and a target height.
  - `SaplingBuilder::add_spend` now takes `extsk` by reference. Also, it no
    longer takes a `diversifier` argument as the diversifier may be obtained
    from the note.
  - `SaplingBuilder::add_output` now takes an `Option<[u8; 512]>` memo instead
    of a `MemoBytes`.
  - `SaplingBuilder::build` no longer takes a prover, proving context, progress
    notifier, or target height. Instead, it has `SpendProver, OutputProver`
    generic parameters and returns `(UnauthorizedBundle, SaplingMetadata)`. The
    caller can then use `Bundle::<InProgress<Unproven, _>>::create_proofs` to
    create spend and output proofs for the bundle.
  - `Error` has new error variants:
    - `Error::DuplicateSignature`
    - `Error::InvalidExternalSignature`
    - `Error::MissingSignatures`
- `sapling_crypto::bundle`:
  - `Bundle` now has a second generic parameter `V`.
  - `Bundle::value_balance` now returns `&V` instead of
    `&zcash_primitives::transaction::components::Amount`.
  - `Authorized::binding_sig` now has type `redjubjub::Signature<Binding>`.
  - `Authorized::AuthSig` now has type `redjubjub::Signature<SpendAuth>`.
  - `SpendDescription::temporary_zcashd_from_parts` now takes `rk` as
    `redjubjub::VerificationKey<SpendAuth>` instead of
    `sapling_crypto::redjubjub::PublicKey`.
  - `SpendDescription::rk` now returns `&redjubjub::VerificationKey<SpendAuth>`.
  - `SpendDescriptionV5::into_spend_description` now takes `spend_auth_sig` as
    `redjubjub::Signature<SpendAuth>` instead of
    `sapling_crypto::redjubjub::Signature`.
  - `testing::arb_bundle` now takes a `value_balance: V` argument.
  - `MapAuth` trait methods now take `&mut self` instead of `&self`.
- `sapling_crypto::circuit::ValueCommitmentOpening::value` is now represented as
  a `NoteValue` instead of as a bare `u64`.
- `sapling_crypto::keys`:
  - `DecodingError` has a new variant `UnsupportedChildIndex`.
  - `ExpandedSpendingKey.ask` now has type `SpendAuthorizingKey`.
  - `ProofGenerationKey.ak` now has type `SpendValidatingKey`.
  - `ViewingKey.ak` now has type `SpendValidatingKey`.
- `sapling_crypto::note_encryption`:
  - `SaplingDomain` no longer has a `P: zcash_primitives::consensus::Parameters`
    type parameter.
  - The following methods now take a `Zip212Enforcement` argument instead of a
    `P: zcash_primitives::consensus::Parameters` argument:
    - `plaintext_version_is_valid`
    - `try_sapling_note_decryption`
    - `try_sapling_compact_note_decryption`
    - `try_sapling_output_recovery_with_ock`
    - `try_sapling_output_recovery`
  - `SaplingDomain::Memo` now has type `[u8; 512]` instead of
    `zcash_primitives::memo::MemoBytes`.
  - `sapling_note_encryption` now takes `memo` as a `[u8; 512]` instead of
    `zcash_primitives::memo::MemoBytes`.
  - The following methods now return `[u8; 512]` instead of
    `zcash_primitives::memo::MemoBytes`:
    - `try_sapling_note_decryption`
    - `try_sapling_output_recovery_with_ock`
    - `try_sapling_output_recovery`
- `sapling_crypto::util::generate_random_rseed` now takes a `Zip212Enforcement`
  argument instead of a `P: zcash_primitives::consensus::Parameters` argument
  and a height.
- `sapling_crypto::value`:
  - `TrapdoorSum::into_bsk` now returns `redjubjub::SigningKey<Binding>` instead
    of `sapling_crypto::redjubjub::PrivateKey`.
  - `CommitmentSum::into_bvk` now returns `redjubjub::VerificationKey<Binding>`
    instead of `sapling_crypto::redjubjub::PublicKey`.

### Removed
- `sapling_crypto::bundle`:
  - `SpendDescription::{read, read_nullifier, read_rk, read_spend_auth_sig}`
  - `SpendDescription::{write_v4, write_v5_without_witness_data}`
  - `SpendDescriptionV5::read`
  - `OutputDescription::read`
  - `OutputDescription::{write_v4, write_v5_without_proof}`
  - `OutputDescriptionV5::read`
- `sapling_crypto::note_encryption::SaplingDomain::for_height` (use
  `SaplingDomain::new` instead).
- `sapling_crypto::redjubjub` module (use the `redjubjub` crate instead).
- `sapling_crypto::spend_sig` (use `redjubjub::SigningKey::{randomize, sign}`
  instead).

## [0.0.1] - 2017-12-06
Initial release to reserve crate name.