---
title: "Useful Groovy Scripts for Jenkins"
tags:
  - jenkins
  - scripts
  - groovy
---

Various Jenkins groovy scripts which can be useful in different situations.

{% include toc %}

### Restart all failed jobs in folder
```
import com.cloudbees.hudson.plugins.folder.AbstractFolder

def folderName = 'molecule/develop'

def restartFailed = {
  if(!it.isBuilding() and !hudson.model.Result.SUCCESS.equals(it.lastBuild.getResult())) {
      build = it.lastBuild
      paramsActions = build?.actions.find{ it instanceof ParametersAction }
      cause = new hudson.model.Cause.UpstreamCause(build)
      causeAction = new hudson.model.CauseAction(cause)
      hudson.model.Hudson.instance.queue.schedule(it, 0, causeAction, paramsActions)
      println "${it.name}: restarted"
  }
}

def doAllItemsInFolder(folderName, closure) {
    AbstractFolder folder = Jenkins.instance.getAllItems(AbstractFolder.class).find {folder -> folderName == folder.fullName };
    folder.getAllJobs()
            .findAll {job -> job.isBuildable()}
            .each {closure(it)};
}

doAllItemsInFolder(folderName, restartFailed);

return 0;
```

### Run log rotation for all jobs
```
def jobs = Hudson.instance.items

jobs.findAll{it.logRotator}.each {
    println("JOB : "+ it.name)
    try {
        println(it.logRotator.perform(it))
    } catch (Exception e) {
        println("rotation failed: " + e)
    }
}
```

### Update jobs' description when job has parameter with specific name
```
import hudson.model.*

def paramName = "name"

for(item in Hudson.instance.items) {
  prop = item.getProperty(ParametersDefinitionProperty.class)
  if(prop != null) {
    for(param in prop.getParameterDefinitions()) {
      try {
        value = param.getDefaultParameterValue().value
        if (param.name == ${paramName}) {
          println("--- " + item.name + " ---")
          item.setDescription("<h3><a href=http://" + value + ":8080/>Link</a></h3>")
        }
      }
      catch(Exception e) {
        println("=== " + param.name + " ===")
      }
    }
  }
}
```

### Rename job by adding prefix to their names
```
import hudson.model.*;

def JOB_PATTERN = ~/^pattern$/;
def PREFIX = "prefix"

(Hudson.instance.items.findAll { job -> job.name =~ JOB_PATTERN }).each { job_to_update ->
    println ("Updating job: " + job_to_update.name);
    def new_job_name = PREFIX + job_to_update.name;
    println ("New name: " + new_job_name);
    job_to_update.renameTo(new_job_name);
    println ("Updated name: " + job_to_update.name);
    println("="*80);
}
```

### Add StringParameter option to all jobs matched regex
```
import hudson.model.*

jobs_regex  = '^pattern$'
key = 'key'
value = 'value'
desc = 'description'

def matchedJobs = Jenkins.instance.items.findAll { job -> job.name =~ /${jobs_regex}/ }

for(job in matchedJobs) {

    println("[ " + job.name + " ] setting " + key + "=" + value)

    newParam = new StringParameterDefinition(key, value, desc)
    paramDef = job.getProperty(ParametersDefinitionProperty.class)

    if (paramDef == null) {
        newArrList = new ArrayList<ParameterDefinition>(1)
        newArrList.add(newParam)
        newParamDef = new ParametersDefinitionProperty(newArrList)
        job.addProperty(newParamDef)
    }
    else {
        found = paramDef.parameterDefinitions.find{ it.name == key }
        if (found == null) {
	        println("[ " + job.name + " ] update key value to '" + value + "'")
            paramDef.parameterDefinitions.add(newParam)
        }
    }
    job.save()
    println()
}
```

### Find outdated jobs
```
import jenkins.model.Jenkins
import groovy.time.TimeCategory

def jobs_regex  = '^pattern'
def max_age_days = 7

def now = new Date()
def matchedJobs = Jenkins.instance.items.findAll { job -> job.name =~ /${jobs_regex}/ }

for(job in matchedJobs) {

  def last_build_time = 'none'
  def days = 0
  def last_build

  try {
    last_build=job.getLastBuild()
  }
  catch (Exception ex) {
    last_build = null
  }

  if (last_build != null){
    last_build_time = last_build.getTime().format("YYYY-MMM-dd HH:MM:SS")
    def diff = TimeCategory.minus(now,last_build.getTime());
    days = diff.days
    user = last_build.getCause(Cause.UserIdCause).getUserId()
  }
  if ( days > max_age_days) {
    printf("%-40s%-40s%-40s%d\n", job.name, last_build_time, user, days)
  }

}
```
