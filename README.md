# jbehave-story-meta-inheritance

This projects tries to add more information to https://jbehave.atlassian.net/browse/JBEHAVE-1134 and the Pull Request made by Valery Yatsynovich at https://github.com/jbehave/jbehave-core/pull/105

There are three stories in this project:
* `story_meta.story` without Examples table and inherits Meta
* `story_with_examples_empty_meta.story` with Examples table and EMPTY Meta in Examples. This one works with the PR applied.
* `story_with_examples_meta.story` with Examples table and with Meta values in Examples. This one does not work even with the PR applied. Last section covers this case.

Let's focus on the first two ones. These two different behaviors between NormalPerformableScenario and ExamplePerformableScenario lead into confusion. One user could have Meta tags inherited using a normal scenario, and later add an Examples table, loosing the inheritance. That's why I think we should inherit them for all scenarios.

This project will run the tests with `+author rjimenez` meta filters.

## Run using JBehave 4.1-SNAPSHOT without any PR applied

From a terminal, just execute:

`mvn clean test`

The output will look like:

```
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.github.rjimgal.jbehave.tests.TestRunner
Jan 15, 2016 11:07:34 AM org.springframework.context.support.AbstractApplicationContext prepareRefresh
INFO: Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@e580929: startup date [Fri Jan 15 11:07:34 CET 2016]; root of context hierarchy
Jan 15, 2016 11:07:34 AM org.springframework.beans.factory.support.DefaultListableBeanFactory preInstantiateSingletons
INFO: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@185d8b6: defining beans [org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,testConfig,org.springframework.context.annotation.ConfigurationClassPostProcessor$ImportAwareBeanPostProcessor#0,inputSteps,propertySourcesPlaceholderConfigurer]; root of factory hierarchy
Annotation interface org.jbehave.core.annotations.UsingSteps not found in class org.github.rjimgal.jbehave.tests.TestRunner
Processing system properties {}
Using controls EmbedderControls[batch=false,skip=false,generateViewAfterStories=true,ignoreFailureInStories=false,ignoreFailureInView=false,verboseFailures=false,verboseFiltering=false,storyTimeouts=300,threads=1,failOnStoryTimeout=false]
Meta[properties={}] excluded by filter '+author rjimenez'
Meta[properties={}] excluded by filter '+author rjimenez'
Meta[properties={tag=value}] excluded by filter '+author rjimenez'
Meta[properties={tag=anotherValue}] excluded by filter '+author rjimenez'
Running story stories/story_meta.story
hi there!
Using timeout for story story_meta.story of 300 secs.
Running story stories/story_with_examples_empty_meta.story
Running story stories/story_with_examples_meta.story
Generating reports view to '/Users/rjimenez/github/jbehave-story-meta-inheritance.standalone/target/jbehave' using formats '[]' and view properties '{navigator=ftl/jbehave-navigator.ftl, views=ftl/jbehave-views.ftl, reports=ftl/jbehave-reports.ftl, nonDecorated=ftl/jbehave-report-non-decorated.ftl, decorated=ftl/jbehave-report-decorated.ftl, maps=ftl/jbehave-maps.ftl}'
Reports view generated with 0 stories (of which 0 pending) containing 0 scenarios (of which 0 pending)
```


## Run using JBehave 4.1-SNAPSHOT with original PR applied

From a terminal, just execute:

`mvn clean test`

The output will look like:

```
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.github.rjimgal.jbehave.tests.TestRunner
Jan 15, 2016 9:30:53 AM org.springframework.context.support.AbstractApplicationContext prepareRefresh
INFO: Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@e580929: startup date [Fri Jan 15 09:30:53 CET 2016]; root of context hierarchy
Jan 15, 2016 9:30:53 AM org.springframework.beans.factory.support.DefaultListableBeanFactory preInstantiateSingletons
INFO: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@185d8b6: defining beans [org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,testConfig,org.springframework.context.annotation.ConfigurationClassPostProcessor$ImportAwareBeanPostProcessor#0,inputSteps,propertySourcesPlaceholderConfigurer]; root of factory hierarchy
Annotation interface org.jbehave.core.annotations.UsingSteps not found in class org.github.rjimgal.jbehave.tests.TestRunner
Processing system properties {}
Using controls EmbedderControls[batch=false,skip=false,generateViewAfterStories=true,ignoreFailureInStories=false,ignoreFailureInView=false,verboseFailures=false,verboseFiltering=false,storyTimeouts=300,threads=1,failOnStoryTimeout=false]
Running story stories/story_meta.story
hi there!
Using timeout for story story_meta.story of 300 secs.
Running story stories/story_with_examples_empty_meta.story
hi there!
bye bye
Running story stories/story_with_examples_meta.story
Meta[properties={tag=value}] excluded by filter '+author rjimenez'
Meta[properties={tag=anotherValue}] excluded by filter '+author rjimenez'
Generating reports view to '/Users/rjimenez/github/jbehave-story-meta-inheritance.standalone/target/jbehave' using formats '[]' and view properties '{navigator=ftl/jbehave-navigator.ftl, views=ftl/jbehave-views.ftl, reports=ftl/jbehave-reports.ftl, nonDecorated=ftl/jbehave-report-non-decorated.ftl, decorated=ftl/jbehave-report-decorated.ftl, maps=ftl/jbehave-maps.ftl}'
Reports view generated with 0 stories (of which 0 pending) containing 0 scenarios (of which 0 pending)
```

## Scenario not fixed by the original PR

The last story, i.e. `story_with_examples_meta.story` is not fixed by the original PR.

When Scenario starts to be performed, in `PerformableTree:983` we have the following check:

`if (!parameterMeta.isEmpty() && !context.filter().allow(parameterMeta)) {`

Since for this case we do not have an empty parameterMeta, the second condition is evaluated, and since parameterMeta only contains the Meta values from Examples table, it will not be allowed.

### A new PR to fix this special scenario

https://github.com/jbehave/jbehave-core/pull/110

Unfortunately, Scenario Meta object does not contain Meta from Story level, we are inheriting them only to check, wether scenario should be added to PerformableTree or not.

We could store inherited Meta into Scenario and Examples level, however this may confuse people, specially when it comes to reporting, and to know from which level comes certain Meta.

For that reason, I am adding a new Meta from Story into Scenario model.

However, I am not too happy with the proposed new PR. Maybe we should try to think bigger and add a relationship between Scenario and the Story where it is located?

After running with this new PR, output looks like:

```
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.github.rjimgal.jbehave.tests.TestRunner
Jan 15, 2016 11:52:51 AM org.springframework.context.support.AbstractApplicationContext prepareRefresh
INFO: Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@e580929: startup date [Fri Jan 15 11:52:51 CET 2016]; root of context hierarchy
Jan 15, 2016 11:52:51 AM org.springframework.beans.factory.support.DefaultListableBeanFactory preInstantiateSingletons
INFO: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@185d8b6: defining beans [org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,testConfig,org.springframework.context.annotation.ConfigurationClassPostProcessor$ImportAwareBeanPostProcessor#0,inputSteps,propertySourcesPlaceholderConfigurer]; root of factory hierarchy
Annotation interface org.jbehave.core.annotations.UsingSteps not found in class org.github.rjimgal.jbehave.tests.TestRunner
Processing system properties {}
Using controls EmbedderControls[batch=false,skip=false,generateViewAfterStories=true,ignoreFailureInStories=false,ignoreFailureInView=false,verboseFailures=false,verboseFiltering=false,storyTimeouts=300,threads=1,failOnStoryTimeout=false]
Running story stories/story_meta.story
hi there!
Using timeout for story story_meta.story of 300 secs.
Running story stories/story_with_examples_empty_meta.story
hi there!
bye bye
Running story stories/story_with_examples_meta.story
hi there!
bye bye
Generating reports view to '/Users/rjimenez/github/jbehave-story-meta-inheritance.standalone/target/jbehave' using formats '[]' and view properties '{navigator=ftl/jbehave-navigator.ftl, views=ftl/jbehave-views.ftl, reports=ftl/jbehave-reports.ftl, nonDecorated=ftl/jbehave-report-non-decorated.ftl, decorated=ftl/jbehave-report-decorated.ftl, maps=ftl/jbehave-maps.ftl}'
Reports view generated with 0 stories (of which 0 pending) containing 0 scenarios (of which 0 pending)
```
