# Myriad Library builder

Builds and checks releases of open-source addons for the [Myriad Library](https://myriadclient.dev/library), in the
open: anyone can read this workflow and every run's log.

The Library's API starts [`build.yml`](.github/workflows/build.yml) for one submission when a developer publishes a
release of an addon that has the Myriad Library GitHub App installed. The run:

1. **prepare**: checks the repository and tag with GitHub, fetches the source at the tag's commit, hashes its build
   logic, and checks the addon's listing back to Myriad's root keys;
2. **build**: builds the tagged source with `./gradlew jar` on JDK 25, with no permissions and no secrets;
3. **inspect**: downloads the developer's released jar and `.sig` and looks through it without running it;
4. **sign**: signs the release for the Library with the version key, only if its build logic is unchanged since its
   listing was reviewed, the released jar is byte for byte what its source builds, its developer signed it, and the
   inspection found nothing the maintainer hasn't accepted;
5. **report**: tells the Library how it went (the developer sees it as a "Myriad Library" check on the release).

Anything that isn't signed waits for the Myriad maintainer's review. New addons always do. The steps' logic is in
Myriad's signer jar (`dev.myriadclient.gradle.signing` in
[myriad-core](https://github.com/MyriadClient/myriad-core/tree/master/gradle-plugin/src/main/java/dev/myriadclient/gradle/signing)),
pinned here by its SHA-256.

Closed-source addons are built in a private repository instead, and only ever signed by a person.

## Setup (maintainer)

- Environment `library-sign`: deployment branch `main` only; secret `LIBRARY_VERSION_KEY` (`ed25519-private:...`).
- Repository secret `LIBRARY_BUILDER_TOKEN` (its SHA-256 is the API's `LIBRARY_BUILDER_OPEN_TOKEN_SHA256`).
- `main` protected by a ruleset: no force pushes, only the maintainer pushes. No `pull_request` workflows here, ever.
- Organisation: actions restricted to GitHub's own plus the pinned ones used here, full-SHA pinning required.
