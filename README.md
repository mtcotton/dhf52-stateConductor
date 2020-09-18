# dhf52-stateConductor

## Steps

1. Create gradle wrapper
    ```
    ~/Downloads/gradle-5.5/bin/gradle wrapper --gradle-version 4.6
    ```
1. Create `build.gradle`
    ```groovy
    plugins {
        id "net.saliman.properties" version "1.5.1"
        id "com.marklogic.ml-data-hub" version "5.2.4"
    }
    ```
1. Initialize Hub
    ```
    ./gradlew hubInit
    ```
1. Create `.gitignore`
    ```
    gradle-local.properties
    ```
1. Enter credentials to `gradle-local.properties`
1. Update `gradle.properties`
   * Replace `801` with `???`
   * Replace `data-hub-` with `???-`
1. Update project structure
   * Rename `src/main/ml-config/databases/data-hub-staging-SCHEMAS` to `???-staging-SCHEMAS`
1. Create a flow
   * `./gradlew hubCreateEntity -PentityName=Person`
   * `./gradlew hubCreateFlow -PflowName=Person`
     * Update Step 1 `inputFilePath`, `outputURIReplacement`, `collections`
     * Update Step 2 `sourceQuery`, `collections`, `targetEntity`, `mapping`
   * Create mapping
    ```json
    {
        "lang" : "zxx",
        "name" : "person-sample",
        "description" : "",
        "version" : 1,
        "targetEntityType" : "Person-0.0.1/Person",
        "sourceContext" : "//",
        "sourceURI" : "",
        "properties" : {
            "id" : {
            "sourcedFrom" : "id"
            },
            "firstName" : {
            "sourcedFrom" : "first"
            },
            "lastName" : {
            "sourcedFrom" : "last"
            }
        }
    }
    ```
   * `./gradlew mlLoadModules`
   * `./gradlew hubRunFlow -PflowName=person -Psteps="1"`
1. Add state conductor dependencies
    ```groovy
    repositories {
        jcenter()
        mavenLocal()
        maven {url "http://developer.marklogic.com/maven2/"}
    }

    dependencies {
        mlBundle "com.marklogic:marklogic-state-conductor:0.7.0"
    }
    ```
1. Add state machine `src/main/ml-data/stateMachines/personMapping.asl.json`
    ```json
    {
        "Comment": "Calls mapping step of `person` flow",
        "mlDomain": {
            "context": []
        },
        "StartAt": "runMappingStep",
        "States": {
            "runMappingStep": {
                "Type": "Task",
                "Comment": "runs the mapping step",
                "Resource": "/state-conductor/actions/common/dhf/dhf5RunFlowStepAction.sjs",
                "Parameters": {
                    "name": "person",
                    "step": 2,
                    "flowOptions": {
                    }
                },
                "Next": "Success",
                "Catch": [
                    {
                        "ErrorEquals": [
                            "States.ALL"
                        ],
                        "Next": "FailState"
                    }
                ]
            },
            "Success": {
                "Type": "Succeed"
            },
            "FailState": {
                "Type": "Fail"
            }
        }
    }
    ```
1. 