#Spotless Android

##Introduction

Since our project has both JAVA and KOTLIN files, we thought of going with spotless for code linting and formatting, as it supports multiple languages(Java,Kotlin,XML etc).
But due to some reasons ,it is not formatting kt files as expected. So we have decided to integrate both Spotless(for java and XML files) and ktlint (for Kotlin files).

Spotless - https://github.com/diffplug/spotless ktlint - https://github.com/pinterest/ktlint

##Requirement

1. Spotless requires JRE 8+ and less than 15.
2. Since our ASDA gradle version is 5.1.1 so we can use , id 'com.diffplug.gradle.spotless' version '4.5.1' which supports all the way back to Gradle 2.x`.

##Integration with project (As a project level)

1.  First of all, add the following dependency to your project-level build.gradle:

          ```
          classpath ‘com.diffplug.spotless:spotless-plugin-gradle:4.5.1’

          ```

2. Create a dedicated gradle file and apply it to the project one. It is as simple as creating a spotless.gradle file (you name it). Please refer below file

```
   apply plugin: "com.diffplug.gradle.spotless"

                 spotless {

                     java {
                         target '**/*.java'
                         googleJavaFormat()
                         removeUnusedImports()
                         trimTrailingWhitespace()
                         indentWithSpaces()
                         endWithNewline()
                     }

                     format 'misc', {
                         target '**/*.gradle', '**/*.md', '**/.gitignore'
                         indentWithSpaces()
                         trimTrailingWhitespace()
                         endWithNewline()
                     }

                     format 'xml', {
                       target project.fileTree(project.rootDir) {
                                   include '**/*.xml'
                                   exclude '**/.idea/*.*'
                               }
                               indentWithSpaces()
                               trimTrailingWhitespace()
                               endWithNewline()
                     }
                 }
  ```

 3. Add below line on your project level build.gradle file

          ```
          apply from: './spotless.gradle'
          ```

 4. Configure ktlint in project level. Add below changes in project level.

         ```
         configurations {
            ktlint
         }

        ktlint "com.pinterest:ktlint:0.41.0"
        ```

 5. Finally add a ktlint linting and formatting gradle tasks in project level build.gradle (which will be executed as a terminal commands)
        ```
         // task for liting the kt files
         task ktlint(type: JavaExec, group: "verification") {
             description = "Check Kotlin code style."
             classpath = configurations.ktlint
             main = "com.pinterest.ktlint.Main"
             args "src/**/*.kt"
         }

         check.dependsOn ktlint

         // task for auto-formatting the kt files

         task ktlintFormat(type: JavaExec, group: "formatting") {
             description = "Fix Kotlin code style deviations."
             classpath = configurations.ktlint
             main = "com.pinterest.ktlint.Main"
             args "-F", "src/**/*.kt"
         }
         ```
 6. Sync the project, Now spotless and ktlint has successfully been integrated in ouR project as project level.

 7. Execute below gradle commands to lint the code base

    ```
    ./gradlew spotlessCheck

    ./gradlew ktlint
    ```

 8. Execute below commands to format and fix the linting errors
    ```
    ./gradlew spotlessApply

    ./gradlew ktlintFormat
    ```

 9. If you follow above steps to format and fix the linting issues, you will get around 800 file changes which will bloat your PR and hard for reviewers to review the changes.
    so we have decided to apply the rules as module level for each modules separately. In the next section we have explained how to achieve this.


 ##Integration with project (As a module level)

 1. Follow 1 and 2 points as it is from previous section, form 3rd point onwards, apply those same changes in module level build.gradle file and sync.

 2. Execution of the commands will differ a bit as we need to move to respective module directory on the terminal to execute the linting and formatting gradle commands.


 ##Note

 1. After formatting the files in module level, while creating a PR skip the gradle changes and commit only formatting
    changes to the PR. (Since this should be added as a project level once after formatting whole project)

