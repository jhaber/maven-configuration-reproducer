# maven-configuration-reproducer

During Maven 3.9.0 upgrade we noticed a change in behavior in how configuration is parsed. It seems to manifest when you try to clear plugin configuration,  while also reconfiguring that same plugin inside a profile (in the same pom). The setup in this repo is:
- `parent` module configures the shade plugin with an `outputFile` parameter
- `child` module clears this configuration (using `<configuration combine.self="override"/>`)
- `child` module also has a profile which configures the shade plugin (adding a `finalName` parameter)
- `nested-child` module activates this profile

It is expected that `child` and `nested-child` do not inherit the `outputFile` parameter, because the shade configuration is removed in `child` pom.xml. This works in Maven 3.8.7, but in Maven 3.9.0 the `nested-child` module does inherit the `outputFile` parameter. I assume this has something to do with the profile getting activated and causing the configuration to not get merged properly, but I'm not sure. 

I also built a custom version of Maven 3.9.0 which uses plexus-utils 3.3.1, and the behavior is back to normal. So I think the behavior change is in plexus-utils (although it could be that Maven is "mis-using" plexus-utils, and the plexus-utils change is technically correct). I tested with a few different plexus-utils versions and it seems like the behavior change was introduced in plexus-utils 3.4.0. 

### Observed behavior with Maven 3.8.7 (which uses plexus-utils 3.3.1)

Checking `parent` module configuration with:  
`apache-maven-3.8.7/bin/mvn help:effective-pom -N | grep outputFile | wc -l`  
✅ Prints `1`, meaning the configuration contains an `outputFile` parameter, which is expected  

Checking `child` module configuration with:  
`apache-maven-3.8.7/bin/mvn help:effective-pom -pl child | grep outputFile | wc -l`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking `nested-child` module configuration with:  
`apache-maven-3.8.7/bin/mvn help:effective-pom -pl child/nested-child | grep outputFile | wc -l`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking all modules with:  
`apache-maven-3.8.7/bin/mvn help:effective-pom | grep outputFile | wc -l`  
✅ Prints `1`, meaning only `parent` module contains an `outputFile` parameter, which is expected  

### Observed behavior with Maven 3.9.0 (which uses plexus-utils 3.4.2)

Checking `parent` module configuration with:  
`apache-maven-3.9.0/bin/mvn help:effective-pom -N | grep outputFile | wc -l`  
✅ Prints `1`, meaning the configuration contains an `outputFile` parameter, which is expected  

Checking `child` module configuration with:  
`apache-maven-3.9.0/bin/mvn help:effective-pom -pl child | grep outputFile | wc -l`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking `nested-child` module configuration with:  
`apache-maven-3.9.0/bin/mvn help:effective-pom -pl child/nested-child | grep outputFile | wc -l`  
❌ Prints `1`, meaning the configuration contains an `outputFile` parameter, which is NOT expected  

Checking all modules with:  
`apache-maven-3.9.0/bin/mvn help:effective-pom | grep outputFile | wc -l`  
❌ Prints `2`, meaning `parent` and `nested-child` modules both contain an `outputFile` parameter, which is NOT expected 

### Observed behavior with Maven 3.9.0 (built against plexus-utils 3.3.1)

Checking `parent` module configuration with:  
`apache-maven-3.9.0-plexus-3.3.1/bin/mvn help:effective-pom -N | grep outputFile | wc -l`  
✅ Prints `1`, meaning the configuration contains an `outputFile` parameter, which is expected  

Checking `child` module configuration with:  
`apache-maven-3.9.0-plexus-3.3.1/bin/mvn help:effective-pom -pl child | grep outputFile | wc -l`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking `nested-child` module configuration with:  
`apache-maven-3.9.0-plexus-3.3.1/bin/mvn help:effective-pom -pl child/nested-child | grep outputFile | wc -l`  
✅ Prints `0`, meaning the configuration does not contain an `outputFile` parameter, which is expected  

Checking all modules with:  
`apache-maven-3.9.0-plexus-3.3.1/bin/mvn help:effective-pom | grep outputFile | wc -l`  
✅ Prints `1`, meaning only `parent` module contains an `outputFile` parameter, which is expected 
