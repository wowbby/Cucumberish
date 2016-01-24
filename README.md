# Cucumberish
Cucumberish is a native Objective-C framework for Behaviour Driven Development inspired by the amazing way of writing automated test cases introduced originally by Cucumber.
For those who do not know what it means: Simply this framework will help developers and non-developers to write test cases in plain English and in a very organized manner.
# Features

* Full integration with Xcode Test Navigator.
* Can be used with continues integration servers, or cloud services; just like XC unit tests!
* When any test fail, Xcode Test Navigator will point to your .feature file on the line that failed.
* Your test reports will appear in your Reports Navigator just like any XC unit tests.
* No Ruby, "command line-only", or any other non-ios related languages or chaotic dependency chain needed to install or use Cucumberish at all!
* Few minutes installation!
* Step implementations are done directly in Objective-C or Swift! Which means you can breakpoint and debug your steps implementations easily.
* Can be used with Unit Test as well as UI Test targets!

##### Here is a quick animated GIF that shows you how it ends up in action

![Cucumberish In Action](https://cloud.githubusercontent.com/assets/5157350/12533701/512a8cd8-c239-11e5-9636-1152353aa47f.gif)

# Installation
You can install Cucumberish manually, in no more than few minutes, or using cocoapods.
If you will use Cucumberish with UI Test target, you should use the manual installation; for some reasons cocoapods fails to copy the required resources with UI Test targets.

### Manual Installation
1. Copy the content of Cucumberish folder into your project and add it to your test target.
2. This step is super important for proper reporting. Go to your test target build settings, and add the following preprocess macro:

    ```
    SRC_ROOT=$(SRCROOT)
    ```

3. And that's it for including Cucumberish in your project!


### Cocoapods Installation
1. Add Cucumberish pod to your podfile to be added to your test target and run pod install.

    ```
    target 'YourTestTargetName' do
        pod 'Cucumberish'
    end
    ```
### Post installation steps:
1. Go to your test target folder and create a subfolder. Let's call it **Features**.
2. Add this folder to your test target in Xcode as a Folder, **not** a group! This is a very important step.
![Features Folder as Folder not a Group](https://cloud.githubusercontent.com/assets/5157350/12533357/f7a94448-c22d-11e5-904a-1c353a76d604.png)
3. Inside this folder, you will create the .feature files which will contain your test's features and scenarios.
    - ##### For Objective-C test targets:
        - When you create a test target, Xcode creates a test case file for you. Open this file and replace its content with the following:
        
            ```Objective-C
            #import "Cucumberish.h"
            //#import <Cucumberish/Cucumberish.h> if installed using cocoapods
            __attribute__((constructor))
            void CucumberishInit()
            {
                //Define your step implememntations (the example project contains set of basic implementations using KIF)
                Given(@"it is home screen", ^void(NSArray *args, id userInfo) {
                    //Step implementation code goes here
                });
                And(@"all data cleared", ^void(NSArray *args, id userInfo) {
                    //Step implementation code goes here
                });
                //Optional step, see the comment on this property for more information
                [Cucumberish instance].fixMissingLastScenario = YES;
                //Tell Cucumber the name of your features folder and let it execute them for you...
                [[[Cucumberish instance] parserFeaturesInDirectory:@"Features" featureTags:nil] beginExecution];
            }
            ```
        
    - ##### For Swift test targets:
        1. Create and add an Objective-C .m file to your test target, when you do this Xcode will prompt you about creating a bridge file, confirm the creation of this file.
        2. In the .m file you just add replace whatever content it has with the following:
            
            ```Objective-C
            //Replace CucumberishExampleUITests with the name of your swift test target
            #import "CucumberishExampleUITests-Swift.h"
            __attribute__((constructor))
            void CucumberishInit()
            {
                [CucumberishInitializer CucumberishSwiftInit];
            }
            ```

        3. In the bridge header file that Xcode created for you in the first step above, add the following import
            ```Objective-C
            #import "Cucumberish.h"
            ```
        4. Last but not least, replace the content of the default .swift test case file that created with your test target with the following:
        
            ```Swift
            import Foundation
            class CucumberishInitializer: NSObject {
                class func CucumberishSwiftInit()
                {
                    var application : XCUIApplication!
                    //A closure that will be executed just before executing any of your features
                    beforeStart { () -> Void in
                        application = XCUIApplication()
                    }
                    //A Given step definition
                    Given("the app is running") { (args, userInfo) -> Void in
                        application.launch()
                    }
                    //Another step definition
                    And("all data cleared") { (args, userInfo) -> Void in
                        //Assume you defined an "I tap on \"(.*)\" button" step previousely, you can call it from your code as well.
                        SStep("I tap the \"Clear All Data\" button")
                    }
                    //Tell Cucumber the name of your features folder and let it execute them for you...
                    Cucumberish.executeFeaturesInDirectory("ExampleFeatures", featureTags: nil)
                }
            }
            ```
            
# Getting started
Now you have Cucumberish in place and you followed all the installation and post-installation instructions; it is time to write a very sample feature and scenario with few steps.
Since the exact step implementations will be very different thing between one project and the other, we will not dig deep into it; we will just see the approach on how to get there. I will assume your test target is an Objective-C one for the sake of demonstration; but the same prenciples can be applied on Swift targets.

Start by creating a new file in your features folder, we will call it example.feature.

_Note:_ You can have only one feature per file, but as many scenarios as you want.

Open this file to edit in Xcode (or any text editor you prefer) and write the followin as your very first feature:
```Gherkin
Feature: Example
# This is a free text description as an inline documentation for your features, you can omit it if you want.
# However, it is very adviseble to well describe your features.
As someone who plans to automate the iOS projet test cases, I will use Cucumberish.

# First scenario is the scenario name, which will also appear in a proper format in Xcode test navigator
Scenario: First scenario

    # This is the first step in the scenario
    # Also noteworthy; a "Given" step, should be treated as the step that defines the app state before going into the rest of the scenario
    # Or consider it as a precondition for the scenario;
    # For example to post a comment, the user most be logged in, then you should say something similar to "Given the user is logged in" as your Given step.
    Given I have a very cool app
    
    
    # The grammar being used, is completely defined by you and your QA team; it is up to you to find the best way to define your functionality.
    # Only keep in mind that every step must start with "Given", "When", "Then", "And", and "But".
    When I automate it with "Cucumberish"
    Then I will be more confident about the quality of the project and its releases
```

Now you have one cool feature in place, it is time to implement its stesp. You only need to care about the steps, not the feature or the scenario it self. So your step implementations are done out of any context.

In the body of the CucumberishInit C function and before calling -[Cucumberish beginExecution] method, add the following:

``` Objective-C
    Given(@"I have a very cool app", ^(NSArray<NSString *> *args, NSDictionary *userInfo) {
        //Now it is expected that you will do whatever necessary to make sure "I have a very cool app" condition is satisfied :)
        NSLog(@"\"Given I have a very cool app\" step is implemented");
    });
    
    // The step implementation matching text can be any valid regular expression text
    // Your regex capturing groups will be added to the args array in the same order they have been captured by your regex string
    When(@"^I automate it with \"(.*)\"$", ^(NSArray<NSString *> *args, NSDictionary *userInfo) {
        NSLog(@"I am gonna automate my test cases with \"%@\"", args[0]);
    });
    
    Then(@"Then I will be more confident about the quality of the project and its releases", ^(NSArray<NSString *> *args, NSDictionary *userInfo) {
        NSLog(@"Implemented step for the sake of the example");
    });

```

And that's it! You have implemented your first feature scenario steps!

# Examples
Creating a wiki with as much details as possible is a work in progrees. Meanwhile, clone this repository and open the file CucumberishExample/CucumberishExample.xcworkspace
This workspace is the default workspace you get when you use Cocoapods.
In the CucumberishExampe project there are three targets:

1. Example app target:
    - It is very small app with few screens and easy to understand flow and imeplementation; in the storyboards most of its UI components has accessibility label. You can find the accessibility labels from storyboard directly or in the simulator using the accessibility inspector.
    
2. Unit test target:
    - This target uses [KIF](https://github.com/kif-framework/KIF) to interact with the UI in the steps implementation.
    - It is a very good idea to take a look at the file CucumberishExampleTests/Steps/CCIStepsUsingKIF.m to see many examples for how to define your step implementations in many different ways. This target uses Objective-C
    - While walking through the step imeplementations, see how this implementation is being used in the CucumberishExampleTests/ExampleFeatures .feature files.

3. UI Test Target:
    - While I was not impressed enough by Apple UI automation API and its so much limitions, I decided to add the example target to complete the chain. It has smaller set of step implementation examples, and written in Swift.

Feel free to take this step implementation as a starting point and use them as much as you want; just rememberto choose what fits best with your needs because you will build your own implementations in all cases.
            
# To Be Continued
