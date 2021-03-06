# Uber Apk Signer
A tool that helps signing, [zip aligning](https://developer.android.com/studio/command-line/zipalign.html) and verifying multiple Android application packages (APKs) with either debug or provided release certificates (or multiple). It supports [v1 and v2 Android signing scheme](https://developer.android.com/about/versions/nougat/android-7.0.html#apk_signature_v2). Easy and convenient debug signing with embedded debug keystore. Automatically verifies signature and zipalign after every signing.

[![GitHub release](https://img.shields.io/github/release/patrickfav/uber-apk-signer.svg)](https://github.com/patrickfav/uber-apk-signer/releases/latest)
[![Build Status](https://travis-ci.org/patrickfav/uber-apk-signer.svg?branch=master)](https://travis-ci.org/patrickfav/uber-apk-signer)
[![Coverage Status](https://coveralls.io/repos/github/patrickfav/uber-apk-signer/badge.svg?branch=master)](https://coveralls.io/github/patrickfav/uber-apk-signer?branch=master)

Main features:

* zipalign, (re)signing and verifying of multiple APKs in one step
* verify signature (with hash check) and zipalign of multiple APKs in one step
* built-in zipalign & debug keystore for convenient usage
* supports v1 and v2 android apk singing scheme
* support for multiple signatures for one APK
* crypto/signing code relied upon official implementation

Basic usage:

    java -jar uber-apk-signer.jar --apks /path/to/apks

This should run on any Windows, Mac or Linux machine where Java8+ is installed. 

### Requirements

* JDK 8
* Currently on Linux 32bit: zipalign must be set in `PATH`

## Download

[Grab jar from latest Release](https://github.com/patrickfav/uber-apk-signer/releases/latest)

## Demo

[![asciicast](https://asciinema.org/a/91092.png)](https://asciinema.org/a/91092)

## Command Line Interface

    -a,--apks <file/folder>           Can be a single apk or a folder containing multiple apks. These are used
                                      as source for zipalining/signing/verifying. It is also possible to
                                      provide multiple locations space seperated (can be mixed file folder):
                                      '/apk /apks2 my.apk'. Folder will be checked non-recursively.
       --allowResign                  If this flag is set, the tool will not show error on signed apks, but
                                      will sign them with the new certificate (therefore removing the old
                                      one).
       --debug                        Prints additional info for debugging.
       --dryRun                       Check what apks would be processed without actually doing anything.
    -h,--help                         Prints help docs.
       --ks <keystore>                The keystore file. If this isn't provided, will try to sign with a debug
                                      keystore. The debug keystore will be searched in the same dir as
                                      execution and 'user_home/.android' folder. If it is not found there a
                                      built-in keystore will be used for convenience. It is possible to pass
                                      one or multiple keystores. The syntax for multiple params is
                                      '<index>=<keystore>' for example: '1=keystore.jks'. Must match the
                                      parameters of --ksAlias.
       --ksAlias <alias>              The alias of the used key in the keystore. Must be provided if --ks is
                                      provided. It is possible to pass one or multiple aliases for multiple
                                      keystore configs. The syntax for multiple params is '<index>=<alias>'
                                      for example: '1=my-alias'. Must match the parameters of --ks.
       --ksDebug <keystore>           Same as --ks parameter but with a debug keystore. With this option the
                                      default keystore alias and passwords are used and any arguments relating
                                      to these parameter are ignored.
       --ksKeyPass <password>         The password for the key. If this is not provided, caller will get a
                                      user prompt to enter it. It is possible to pass one or multiple
                                      passwords for multiple keystore configs. The syntax for multiple params
                                      is '<index>=<password>'. Must match the parameters of --ks.
       --ksPass <password>            The password for the keystore. If this is not provided, caller will get
                                      a user prompt to enter it. It is possible to pass one or multiple
                                      passwords for multiple keystore configs. The syntax for multiple params
                                      is '<index>=<password>'. Must match the parameters of --ks.
    -o,--out <path>                   Where the aligned/signed apks will be copied to. Must be a folder. Will
                                      create, if it does not exist.
       --overwrite                    Will overwrite/delete the apks in-place
       --skipZipAlign                 Skips zipAlign process. Also affects verify.
    -v,--version                      Prints current version.
       --verbose                      Prints more output, especially useful for sign verify.
       --verifySha256 <cert-sha256>   Provide one or multiple sha256 in string hex representation (ignoring
                                      case) to let the tool check it against hashes of the APK's certificate
                                      and use it in the verify process. All given hashes must be present in
                                      the signature to verify e.g. if 2 hashes are given the apk must have 2
                                      signatures with exact these hashes (providing only one hash, even if it
                                      matches one cert, will fail).
    -y,--onlyVerify                   If this is passed, the signature and alignment is only verified.
       --zipAlignPath <path>          Pass your own zipalign executable. If this is omitted the built-in
                                      version is used (available for win, mac and linux)

### Examples

Provide your own out directory for signed apks

    java -jar uber-apk-signer.jar -a /path/to/apks --out /path/to/apks/out

Only verify the signed apks

    java -jar uber-apk-signer.jar -a /path/to/apks --onlyVerify

Sign with your own release keystore

    java -jar uber-apk-signer.jar -a /path/to/apks --ks /path/release.jks --ksAlias my_alias

Provide your own zipalign executable

    java -jar uber-apk-signer.jar -a /path/to/apks --zipAlignPath /sdk/build-tools/24.0.3/zipalign

Provide your own location of your debug keystore

    java -jar uber-apk-signer.jar -a /path/to/apks --ksDebug /path/debug.jks

Sign with your multiple release keystores

    java -jar uber-apk-signer.jar -a /path/to/apks --ks 1=/path/release.jks 2=/path/release2.jks --ksAlias 1=my_alias1 2=my_alias2

Use multiple locations or files (will ignore duplicate files)

    java -jar uber-apk-signer.jar -a /path/to/apks /path2 /path3/select1.apk /path3/select2.apk

Provide your sha256 hash to check against the signature:

    java -jar uber-apk-signer.jar -a /path/to/apks --onlyVerify --verifySha256 ab318df27


### Process Return Value

This application will return `0` if every signing/verifing was successful, `1` if an error happens (e.g. wrong arguments) and `2` if at least 1 sign/verify process was not successful.

### Debug Signing Mode

If no keystore is provided the tool will try to automatically sign with a debug keystore. It will try to find on in the following locations (descending order):

* Keystore location provided with `--ksDebug`
* `debug.keystore` in the same directory as the jar executable
* `debug.keystore` found in the `/user_home/.android` folder
* Embedded `debug.keystore` packaged with the jar executable

A log message will indicate which one was chosen.

### Zipalign Executable

[`Zipalign`](https://developer.android.com/studio/command-line/zipalign.html) is a tool developed by Google to optimize zips (apks). It is needed if you want to upload it to the Playstore otherwise it is optional. By default this tool will try to zipalign the apk, therefore it will need the location of the executable. If the path isn't passed in the command line interface, the tool checks if it is in `PATH` environment variable, otherwise it will try to use an embedded version of zipalign. 

If `--skipZipAlign` is passed no executable is needed.

### v1 and v2 Signing Scheme

[Android 7.0 introduces APK Signature Scheme v2](https://developer.android.com/about/versions/nougat/android-7.0.html#apk_signature_v2), a new app-signing scheme that offers faster app install times and more protection against unauthorized alterations to APK files. By default, Android Studio 2.2 and the Android Plugin for Gradle 2.2 sign your app using both APK Signature Scheme v2 and the traditional signing scheme, which uses JAR signing.

[APK Signature Scheme v2 is a whole-file signature scheme](https://source.android.com/security/apksigning/v2.html) that increases verification speed and strengthens integrity guarantees by detecting any changes to the protected parts of the APK. The older jarsigning is called v1 schema.

## Build

Use maven (3.1+) to create a jar including all dependencies

    mvn clean package

## Tech Stack

* Java 8
* Maven

# License

Copyright 2016 Patrick Favre-Bulle

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
