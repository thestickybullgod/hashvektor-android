# Contributing to HashVektor

HashVektor is proprietary software maintained by Planet Archives. Source may be visible in
this repository, but that does not grant permission to copy, modify, redistribute, or reuse
it.

## Before changing code

- Open or reference an issue before starting substantial work.
- Keep changes focused on one feature or defect.
- Follow the existing C#, XAML, MVVM, and Android conventions.
- Preserve the privacy claims in `docs/security-privacy.md`.
- Do not add analytics, advertising identifiers, network access, or runtime permissions
  without an explicit design and policy review.
- Do not commit signing keys, purchase secrets, personal data, device data, or passwords.

## Development

Install the required .NET, MAUI, Android SDK, JDK, and NDK tooling described in
[README.md](README.md).

Before submitting a change:

```powershell
dotnet build HashVektor.Android.slnx
dotnet list src/HashVektor.Mobile package --vulnerable --include-transitive
```

Test the affected flow on a phone or emulator. Security-sensitive changes should also verify
the merged Android manifest and app privacy behavior.

## Contributions

Do not submit a pull request until a maintainer has approved the work. Approved contributors
must ensure that their changes are original and do not include third-party code without the
required licence and notice.

Submitting a contribution does not transfer ownership or grant reuse rights. Planet Archives
determines whether, when, and how a contribution is incorporated.
