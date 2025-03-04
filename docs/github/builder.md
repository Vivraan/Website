# Builder

Building the project as part of a workflow may help to create mind-space and focus on the project itself.

Use [Unity - Builder ](https://github.com/features/actions)
to automatically build Unity projects for different platforms.

## Basic setup

By default, the enabled scenes from the project's settings will be built.

Create or edit the file called `.github/workflows/main.yml` and add a job to it.

#### Personal License

Personal licenses require a one-time manual activation step (per unity version).

Make sure you
[acquire and activate](https://github.com/marketplace/actions/unity-request-activation-file)
your license file and add it as a secret.

Then, define the build step as follows:

```yaml
- uses: webbertakken/unity-builder@<version>
  env:
    UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
  with:
    projectPath: path/to/your/project
    unityVersion: 2020.X.XXXX
    targetPlatform: WebGL
```

#### Professional license

Make sure you have set up these variables in the activation step.

- `UNITY_EMAIL` (should contain the email address for your Unity account)
- `UNITY_PASSWORD` (the password that you use to login to Unity)
- `UNITY_SERIAL` (the serial provided by Unity)

Define the build step as follows:

```yaml
- uses: webbertakken/unity-builder@<version>
  env:
    UNITY_EMAIL: ${{ secrets.UNITY_EMAIL }}
    UNITY_PASSWORD: ${{ secrets.UNITY_PASSWORD }}
    UNITY_SERIAL: ${{ secrets.UNITY_SERIAL }}
  with:
    projectPath: path/to/your/project
    unityVersion: 2020.X.XXXX
    targetPlatform: WebGL
```

That is all you need to build your project.

## Storing the build

To be able to access your built files,
they need to be uploaded as artifacts.
To do this it is recommended to use Github Actions official
[upload artifact action](https://github.com/marketplace/actions/upload-artifact)
after any build action.

By default, Builder outputs it's builds to a folder named `build`.

Example:

```yaml
- uses: actions/upload-artifact@v1
  with:
    name: Build
    path: build
```

Builds can now be downloaded as Artifacts in the Actions tab.

## Caching

In order to make builds run faster, you can cache Library files from previous
builds. To do so simply add Github Actions official
[cache action](https://github.com/marketplace/actions/cache) before any unity steps.

Example:

```yaml
- uses: actions/cache@v1.1.0
  with:
    path: path/to/your/project/Library
    key: Library-MyProjectName-TargetPlatform
    restore-keys: |
      Library-MyProjectName-
      Library-
```

This simple addition could speed up your build by more than 50%.

## Configuration options

Below options can be specified under `with:` for the `unity-builder` action.

#### projectPath

Specify the path to your Unity project to be built.
The path should be relative to the root of your project.

_**required:** `false`_
_**default:** `<your project root>`_

#### unityVersion

Version of Unity to use for building the project.

_**required:** `false`_
_**default:** `2019.2.1f11`_

#### targetPlatform

Platform that the build should target.

Must be one of the [allowed values](https://docs.unity3d.com/ScriptReference/BuildTarget.html) listed in the Unity scripting manual.

_**required:** `true`_

#### buildName

Name of the build. Also the folder in which the build will be stored within `buildsPath`.

_**required:** `false`_
_**default:** `<build_target>`_

#### buildsPath

Path where the builds should be stored.

In this folder a folder will be created for every targetPlatform.

_**required:** `false`_
_**default:** `build`_

#### buildMethod

Custom command to run your build.

There are two conditions for a custom buildCommand:

- Must reference a valid path to a `static` method.
- The class must reside in the `Assets/Editor` directory.

_**example:**_

```yaml
- uses: webbertakken/unity-builder@<version>
  with:
    buildMethod: EditorNamespace.BuilderClassName.StaticBulidMethod
```

_**required:** `false`_
_**default:** Built-in script that will run a build out of the box._

#### versioning

Configure a specific versioning strategy

```yaml
- uses: webbertakken/unity-builder@<version>
  with:
    versioning: Semantic
```

Find the available strategies below:

##### Semantic

Versioning out of the box! **(recommended)**

> Compatible with **all platforms**.  
> Does **not** modify your repository.  
> Requires **zero configuration**.

How it works:

> Generates a version based on [semantic versioning](https://semver.org/).  
> Follows `<major>.<minor>.<patch>` for example `0.17.2`.  
> The latest tag dictates `<major>.<minor>` (defaults to 0.0 for no tag).  
> The number of commits (since the last tag, if any) is used for `<patch>`.

No configuration required.

##### Custom

Allows specifying a custom version in the `version` field. **(advanced users)**

> This strategy is useful when your project or pipeline has some kind of orchestration
> that determines the versions.

##### None

No version will be set by Builder. **(not recommended)**

> Not recommended unless you generate a new version in a pre-commit hook. Manually
> setting versions is error-prone.

#### androidVersionCode

Configure the android `versionCode`.

When not specified, the version code is generated from the version using the `major * 1000000 + minor * 1000 + patch` scheme;

#### androidAppBundle

Set this flag to `true` to build '.aab' instead of '.apk'.

_**required:** `false`_
_**default:** `false`_

#### androidKeystoreName

Configure the android `keystoreName`.

_**required:** `false`_
_**default:** `""`_

#### androidKeystoreBase64

Configure the base64 contents of the android keystore file.

The contents will be decoded from base64 with `echo $androidKeystoreBase64 | base64 --decode > $androidKeystoreName`;

_**required:** `false`_
_**default:** `""`_

#### androidKeystorePass

Configure the android `keystorePass`.

_**required:** `false`_
_**default:** `""`_

#### androidKeyaliasName

Configure the android `keyaliasName`.

_**required:** `false`_
_**default:** `""`_

#### androidKeyaliasPass

Configure the android `keyaliasPass`.

_**required:** `false`_
_**default:** `""`_

#### allowDirtyBuild

Allows the branch of the build to be dirty, and still generate the build.

```yaml
- uses: webbertakken/unity-builder@<version>
  with:
    allowDirtyBuild: true
```

Note that it is generally bad practice to modify your branch
in a CI Pipeline. However there are exceptions where this might
be needed. (use with care).

_**required:** `false`_
_**default:** `false`_

#### customParameters

Custom parameters to configure the build.

Parameters must start with a hyphen (`-`) and may be followed by a value (without hyphen).

Parameters without a value will be considered booleans (with a value of true).

```yaml
- uses: webbertakken/unity-builder@<version>
  with:
    customParameters: -profile SomeProfile -someBoolean -someValue exampleValue
```

_**required:** `false`_
_**default:** `""`_

## Outputs

Below are outputs that can be accessed by using `${{ steps.myBuildStep.outputs.outputName }}`, where `myBuildStep` is the id
of the Builder step, and `outputName` is the name of the output.

#### buildVersion

Returns the version that was generated by Builder, following the strategy configured for `versioning`.

```yaml
- uses: webbertakken/unity-builder@<version>
  id: myBuildStep
- run: echo Project Version: ${{ steps.myBuildStep.outputs.buildVersion }}
```

## Complete example

A complete workflow that builds every available platform could look like this:

```yaml
name: Build project

on:
  pull_request: {}
  push: { branches: [master] }

env:
  UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}

jobs:
  buildForSomePlatforms:
    name: Build for ${{ matrix.targetPlatform }} on version ${{ matrix.unityVersion }}
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        projectPath:
          - path/to/your/project
        unityVersion:
          - 2019.2.11f1
          - 2019.3.0f1
        targetPlatform:
          - StandaloneOSX # Build a macOS standalone (Intel 64-bit).
          - StandaloneWindows # Build a Windows standalone.
          - StandaloneWindows64 # Build a Windows 64-bit standalone.
          - StandaloneLinux64 # Build a Linux 64-bit standalone.
          - iOS # Build an iOS player.
          - Android # Build an Android .apk standalone app.
          - WebGL # WebGL.
          - WSAPlayer # Build an Windows Store Apps player.
          - PS4 # Build a PS4 Standalone.
          - XboxOne # Build a Xbox One Standalone.
          - tvOS # Build to Apple's tvOS platform.
          - Switch # Build a Nintendo Switch player.
    steps:
      - uses: actions/checkout@v2
        with:
          lfs: true
      - uses: actions/cache@v1.1.0
        with:
          path: ${{ matrix.projectPath }}/Library
          key: Library-${{ matrix.projectPath }}-${{ matrix.targetPlatform }}
          restore-keys: |
            Library-${{ matrix.projectPath }}-
            Library-
      - uses: webbertakken/unity-builder@<version>
        with:
          projectPath: ${{ matrix.projectPath }}
          unityVersion: ${{ matrix.unityVersion }}
          targetPlatform: ${{ matrix.targetPlatform }}
      - uses: actions/upload-artifact@v1
        with:
          name: Build
          path: build
```

> **Note:** _Environment variables are set for all jobs in the workflow like this._
