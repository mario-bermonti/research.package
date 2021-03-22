---
layout: single
title: Consent
permalink: /consent
toc: true
---

## Creating the Consent Document Model

### Consent Document Content

First you have to create your consent document model using the ResearchPackage classes.

First, the sections of the consent document ([`RPConsentDocument`](https://pub.dev/documentation/research_package/latest/research_package_model/RPConsentDocument-class.html)) has to be created.
ResearchPackage provides a large list of predefined sections types with corresponding images and animations, plus a "Custom" option to build you own:

- Overview,
- DataUse,
- TimeCommitment,
- StudySurvey,
- Withdrawing,
- DataGathering,
- Privacy,
- StudyTasks,
- Welcome,
- AboutUs,
- Goals,
- Benefits,
- DataHandling,
- Duration,
- YourRights,
- Custom
- UserDataCollection\*
- PassiveDataCollection\*

\*The `UserDataCollection` and `PassiveDataCollection` present a list of items - regarding data collection - where each item contains an explanation of why it is being collected.

> You will see that `.withParams` or `.withIdentifier` constructors are being used. This is needed because of the JSON serialization which requires an empty constructor for all the classes eligible for serialization.

To create a section just instantiate [`RPConsentSection`](https://pub.dev/documentation/research_package/latest/research_package_model/RPConsentSection-class.html) with a type and then specify the summary (short text under the title) and the content which can be accessed after tapping on the "Learn more" button).
The `content` attribute is core piece of information to be shown to the user. In the end of the consent flow, the user has to agree to what is stated in each section's `content` field.

```dart
RPConsentSection overviewSection = RPConsentSection.withParams(RPConsentSectionType.Overview)
..summary = "Welcome to this survey"
..content = "Overview dolor sit amet, consectetur adipiscing elit.";

RPConsentSection dataGatheringSection = RPConsentSection.withParams(RPConsentSectionType.DataGathering)
..summary = "This is a summary for Data Gathering."
..content = "Data Gathering dolor sit amet, consectetur adipiscing elit.";
```

The data collection sections are created a little differently. Below is an example of how a `UserDataCollection` is created. It has a list of data types with `RPDataTypeSection` objects. Those contain the title and the reasoning behind their collection.

```dart
RPConsentSection userDataCollection = RPConsentSection.withParams(RPConsentSectionType.UserDataCollection)
  ..summary = "This study will ask you to provide the information about the following categories:"
  ..dataTypes = [
    RPDataTypeSection("Alcohol",
        "You have to input some info about alcohol in the alcohol card."),
    RPDataTypeSection("Mood", "You have to input some info about your mood in the mood card"),
    RPDataTypeSection("Weight", "You have to input some info about your weight in the weight card"),
    RPDataTypeSection(
        "Medications", "You have to input some info about your medications in the medications card"),
    RPDataTypeSection("Smoking", "You have to input some info about your smoking in the smoking card"),
  ];
```

A consent document in ResearchPackage is represented by [`RPConsentDocument`](https://pub.dev/documentation/research_package/latest/research_package_model/RPConsentDocument-class.html).
Create one by specifying the title and the previously created sections.

```dart
RPConsentDocument myConsentDocument = RPConsentDocument.withParams("Title", [overviewSection, dataGatheringSection]);
```

### Collecting a Consent Signature

An important part of obtaining the consent document is collecting the signature from the study participant.
ResearchPackage comes with a built-in support for this.

First, you have to specify the signature you want to collect by creating a [`RPConsentSignature`](https://pub.dev/documentation/research_package/latest/research_package_model/RPConsentSignature-class.html) object with a unique id. The default one collects the user's full name and an image of the user's signature.

Then this the signature must be added to the consent document, as shown below:

```dart
RPConsentSignature mySignature = RPConsentSignature.withIdentifier("uniqueSignatureID");

myConsentDocument.addSignature(mySignature);
```

### The Consent Task

As described in the [architecture](software-architecture), ResearchPackage uses a task as its core organizational unit.
A [`RPOrderedTask`](https://pub.dev/documentation/research_package/latest/research_package_model/RPOrderedTask-class.html) is a sequence of steps where the sequence can not be changed.

To create a consent task you need two steps:

- [`RPVisualConsentStep`](https://pub.dev/documentation/research_package/latest/research_package_model/RPVisualConsentStep-class.html), which is responsible to guide the user through the different content sections in the consent document
- [`RPReviewConsentStep`](https://pub.dev/documentation/research_package/latest/research_package_model/RPConsentReviewStep-class.html), which is responsible to present the whole consent document to the user for review and then collect the signature.

An example of how these two steps are created is shown below:

```dart
RPVisualConsentStep consentVisualStep = RPVisualConsentStep("visualStepID", myConsentDocument);

RPConsentReviewStep consentReviewStep = RPConsentReviewStep("consentReviewstepID", myConsentDocument)
..reasonForConsent = "By tapping AGREE you can take part in the study"
..text = "Agreed?"
..title = "Consent Review Step Title";
```

Optionally, you can also add a completion step in which you thank the user for joining the study or communicate some other information. This is done by creating a [`RPCompletionStep`](https://pub.dev/documentation/research_package/latest/research_package_model/RPCompletionStep-class.html).

```dart
RPCompletionStep completionStep = RPCompletionStep("completionStepID")
  ..title = "Thank You!"
  ..text = "We saved your consent document";
```

Now that you have all the needed steps, these can be combined into an overal task for obtaining the consent.
This is done by creating a [`RPOrderedTask`](https://pub.dev/documentation/research_package/latest/research_package_model/RPOrderedTask-class.html) with a unique string identifier and the list of the previously defined steps.

```dart
RPOrderedTask consentTask = RPOrderedTask("consentTaskID", [consentVisualStep, consentReviewStep, completionStep]);
```

## Presenting the Task

The next step is to present the consent task.
This is done using the UI library of ResearchPackage.

The [`RPUITask`](https://pub.dev/documentation/research_package/latest/research_package_ui/RPUITask-class.html) class will automatically present the task based on each step objects in the task. It also gives you the possibility to gather the results which is the signature in this case.

This widget returns a full screen [`Scaffold`](https://docs.flutter.io/flutter/material/Scaffold-class.html) widget.
Therefore the recommended usage is to create a route which returns an [`RPUITask`](https://pub.dev/documentation/research_package/latest/research_package_ui/RPUITask-class.html) and then navigate to this route. The minimum example is the following:

```dart
class ConsentDocumentTaskRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return RPUITask(
      task: consentTask,
      onSubmit: () {} // Collecting results, see next section
    );
  }
}
```

## Collecting results

Getting the result from a task is done by passing a callback function to the [`RPUITask`](https://pub.dev/documentation/research_package/latest/research_package_ui/RPUITask-class.html) widget's `onSubmit` parameter.
The onSubmit event is triggered when the user finishes the last step in the the list of steps in the task.

You should create a function which takes an input parameter with [`RPTaskResult`](https://pub.dev/documentation/research_package/latest/research_package_model/RPTaskResult-class.html) type.
A minimum example is the following:

```dart
void resultCallback(RPTaskResult result) {
   // Do whatever you want with the result
   // In this case we are just printing the result's keys
   print(result.results.keys);
}
```

When this function is defined, you just pass it to the task widget when constructed.
This changes the our example above to the following:

```dart
@override
Widget build(BuildContext context) {
   return RPUITask(
      task: task,
      onSubmit: resultCallback,
   );
}
```


## Examples

|  <img src="https://raw.githubusercontent.com/cph-cachet/research.package/gh-pages/assets/images/ConsentOverview.png"> |  <img src="https://raw.githubusercontent.com/cph-cachet/research.package/gh-pages/assets/images/ConsentOverviewLearnMore.png"> |   <img src="https://raw.githubusercontent.com/cph-cachet/research.package/gh-pages/assets/images/ConsentOverview.png">|
|:-:|:-:|:-:|
| <img src="https://raw.githubusercontent.com/cph-cachet/research.package/gh-pages/assets/images/ConsentAboutUs.png">  | <img src="https://raw.githubusercontent.com/cph-cachet/research.package/gh-pages/assets/images/ConsentDataGathering.png">  |  <img src="https://raw.githubusercontent.com/cph-cachet/research.package/gh-pages/assets/images/ConsentPassiveDataCollection.png"> |
| <img src="https://raw.githubusercontent.com/cph-cachet/research.package/gh-pages/assets/images/ConsentUserDataCollection.png">  |  <img src="https://raw.githubusercontent.com/cph-cachet/research.package/gh-pages/assets/images/ConsentReview1.png"> | <img src="https://raw.githubusercontent.com/cph-cachet/research.package/gh-pages/assets/images/ConsentReview2.png">  |
| <img src="https://raw.githubusercontent.com/cph-cachet/research.package/gh-pages/assets/images/ConsentSignature.png">  |   |   |