# maven-configuration-reproducer

During Maven 3.9.0 upgrade we noticed a change in behavior in how configuration is parsed. It seems to manifest when you try to clear plugin configuration, while also reconfiguring that same plugin inside a profile (in the same pom). The setup in this repo is:
- `parent` module configures the shade plugin with an `outputFile` parameter
- `child` module clears this configuration
- `child` module also adds a `finalName` parameter to the shade plugin, inside a profile
- `nested-child` module activates this profile

### Observed behavior with Maven 3.8.7

Checking `parent` module configuration with:  
`apache-maven-3.8.7/bin/mvn help:effective-pom -N | grep outputFile | wc -l`  
✅ Prints `1`, meaning the configuration contains an `outputFile` parameter, which is expected  

Checking `child` module configuration with:  
`apache-maven-3.8.7/bin/mvn help:effective-pom -pl child -N | grep outputFile | wc -l`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking `child` module configuration with:  
`(cd child && ../apache-maven-3.8.7/bin/mvn help:effective-pom -N | grep outputFile | wc -l)`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking `nested-child` module configuration with:  
`apache-maven-3.8.7/bin/mvn help:effective-pom -pl child/nested-child -N | grep outputFile | wc -l`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking `nested-child` module configuration with:  
`(cd child/nested-child && ../../apache-maven-3.8.7/bin/mvn help:effective-pom -N | grep outputFile | wc -l)`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking all modules with:  
`apache-maven-3.8.7/bin/mvn help:effective-pom | grep outputFile | wc -l`  
✅ Prints `1`, meaning only `parent` module contains an `outputFile` parameter, which is expected  

### Observed behavior with Maven 3.9.0

Checking `parent` module configuration with:  
`apache-maven-3.9.0/bin/mvn help:effective-pom -N | grep outputFile | wc -l`  
✅ Prints `1`, meaning the configuration contains an `outputFile` parameter, which is expected  

Checking `child` module configuration with:  
`apache-maven-3.9.0/bin/mvn help:effective-pom -pl child -N | grep outputFile | wc -l`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking `child` module configuration with:  
`(cd child && ../apache-maven-3.9.0/bin/mvn help:effective-pom -N | grep outputFile | wc -l)`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking `nested-child` module configuration with:  
`apache-maven-3.9.0/bin/mvn help:effective-pom -pl child/nested-child -N | grep outputFile | wc -l`  
❌ Prints `1`, meaning the configuration contains an `outputFile` parameter, which is NOT expected  

Checking `nested-child` module configuration with:  
`(cd child/nested-child && ../../apache-maven-3.9.0/bin/mvn help:effective-pom -N | grep outputFile | wc -l)`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking all modules with:  
`apache-maven-3.9.0/bin/mvn help:effective-pom | grep outputFile | wc -l`  
❌ Prints `2`, meaning `parent` and `nested-child` modules both contain an `outputFile` parameter, which is NOT expected  
