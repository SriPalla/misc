To achieve this in Jenkins using Groovy, you'll need to:

1. Parse the input version (e.g., `x.x.x-x`) to extract the base version (`x.x.x`).
2. Filter the list of tags to find those matching the base version with different suffixes (`-x`).
3. Sort these matching tags and pick the latest one.

Here's a step-by-step example:

### Sample List of Docker Tags

Assume you have a list of version tags in the following format:

```groovy
def tags = ["1.0.0-1", "1.0.0-2", "1.0.1-1", "1.2.0-1", "2.0.0-1", "2.1.0-1", "3.0.0-1", "1.0.0-3"]
```

### Groovy Script to Get the Latest Version for a Given Base Version

Here's how you can filter, sort, and find the latest version with the given base version:

```groovy
def getLatestVersionWithSuffix(List<String> tags, String baseVersion) {
    // Define a closure to parse the suffix version
    def parseSuffix = { version ->
        def parts = version.split('-')
        parts.length > 1 ? parts[1].toInteger() : 0
    }

    // Filter tags to include only those matching the base version with suffix
    def matchingTags = tags.findAll { it.startsWith(baseVersion) }

    // Sort matching tags based on the suffix version
    def sortedTags = matchingTags.sort { a, b ->
        def suffixA = parseSuffix(a)
        def suffixB = parseSuffix(b)
        suffixA <=> suffixB
    }

    // Return the last tag in the sorted list (latest version with suffix)
    return sortedTags ? sortedTags[-1] : null
}

// Example usage
def tags = ["1.0.0-1", "1.0.0-2", "1.0.1-1", "1.2.0-1", "2.0.0-1", "2.1.0-1", "3.0.0-1", "1.0.0-3"]
def baseVersion = "1.0.0"
def latestVersionWithSuffix = getLatestVersionWithSuffix(tags, baseVersion)
println "Latest Version with Suffix: ${latestVersionWithSuffix}"
```

### Explanation

1. **Filter Matching Tags**:
   - Use `findAll` to filter the list of tags and retain only those that start with the specified base version (`x.x.x`).

2. **Parse Suffix Version**:
   - Define a closure `parseSuffix` to extract the suffix part from the version string and convert it to an integer for sorting.

3. **Sort Tags**:
   - Use the `sort` method with a custom comparator that compares the suffix parts.

4. **Get the Latest Version with Suffix**:
   - The latest version with the suffix is the last element in the sorted list (`sortedTags[-1]`).

### Using in Jenkins Pipeline

You can integrate this Groovy script into a Jenkins pipeline to dynamically fetch the latest Docker tag with the given base version:

```groovy
pipeline {
    agent any

    stages {
        stage('Get Latest Docker Tag with Suffix') {
            steps {
                script {
                    // Example tags list, in real scenarios this could come from Docker CLI or registry API
                    def tags = ["1.0.0-1", "1.0.0-2", "1.0.1-1", "1.2.0-1", "2.0.0-1", "2.1.0-1", "3.0.0-1", "1.0.0-3"]
                    def baseVersion = "1.0.0"
                    
                    def latestVersionWithSuffix = getLatestVersionWithSuffix(tags, baseVersion)
                    if (latestVersionWithSuffix) {
                        echo "Latest Version with Suffix: ${latestVersionWithSuffix}"
                    } else {
                        error "No matching version tags found."
                    }
                }
            }
        }
    }
}

// Groovy function to get the latest version with suffix
def getLatestVersionWithSuffix(List<String> tags, String baseVersion) {
    def parseSuffix = { version ->
        def parts = version.split('-')
        parts.length > 1 ? parts[1].toInteger() : 0
    }

    def matchingTags = tags.findAll { it.startsWith(baseVersion) }
    def sortedTags = matchingTags.sort { a, b ->
        def suffixA = parseSuffix(a)
        def suffixB = parseSuffix(b)
        suffixA <=> suffixB
    }
    return sortedTags ? sortedTags[-1] : null
}
```

### Explanation in Jenkins Pipeline

1. **Pipeline Definition**:
   - The `pipeline` block defines the Jenkins pipeline.

2. **Stage to Get Latest Docker Tag with Suffix**:
   - The `script` block contains the logic to process the tags and determine the latest version with the given base version and suffix.

3. **Tags List**:
   - For demonstration, the tags list is hardcoded. In real use cases, you would fetch this list from your Docker registry.

4. **Invoke Function**:
   - The `getLatestVersionWithSuffix` function is defined and invoked to determine the latest tag with the given base version.

By following this approach, you can dynamically determine the latest version tag with a specific suffix from a list of Docker tags using Groovy in a Jenkins pipeline.
