# Readme


## README for Services

The mandatory file shall be located in a repo root. Only one `README.md` file should be in a repo root.


## README for iOS and Android Modules


### README Naming

Each file shall be named as `README`.

> **NOTE**
>
> For better navigation, the description of module features can be added in separate files (not obligatory to do it in a README file).

If there is a separate file for each feature, name each file according to the feature it describes.

Example:`messenger-call.md`


### README Location

Each module readme **shall be placed in a module directory root.** Example: `inDriverAndroid/messenger/README`

> **NOTE**
>
> In Android, the `README.md` file is expected to be placed adjacent to the module’s `build.gradle` or `build.gradle.kts` file. All Gradle modules must include a `README.md` file.

If there is a separate file for each feature, create in a module root a `/docs` directory and place there the feature files. Example: `inDriverAndroid/messenger/feature-chat/docs/messenger-call`.


### Module README Template

```plaintext
# Module Name (mandatory section)

Product documentation relevant for the module (if available) — [link to a PRD (product requirement documentation]().


## General Information (mandatory section)

Overview information about the module.


### Main Module Screen (mandatory section)

Description of the module main screen.


### Feature Toggles and Their Values (mandatory section)


### How to Open the Module 

Instructions on how to open the module in the application. What do I need to do in order to get to the module?


### Deep Links

Specify the deep links which are incorporated to the module.


### Module Entry Points

Specify the routes from which the module is opened.


## Module Functionality (mandatory section)

If a module has several features, specify a list of this features.


### Feature Name (mandatory section)


#### Feature Overview (mandatory section)


Add the following information (depending on its availability) for a feature:

* Jira task [links]()

* DeepLink [values]()

* Figma [links]()

* Feature toggles [values]


#### Old Screen (optional section)

Add an old screen image with description.


#### New Screen (optional section)

Add a new screen image with description.


#### How to Get to the Module Feature (mandatory section)

Specify the path to get to the module feature.


####  Feature Functioning (mandatory section)

What does this feature do? Specify the detailed description of the module features, including all details for the feature operation.
```


## Module Readme in a Doc Structure

> **NOTE**
>
> The README file should be added to a separate section on the Documentary Platform called "Modules" or a similar section with an appropriate name.


```plaintext
- name: Modules
  items:
    - name: Wallet
      items:
        - name: Driver
          items:
            - name: Main
              href: ./features/wallet/driver/main/readme.md
            - name: Balance
              href: ./features/wallet/driver/balance/readme.md
            - name: Transaction
              href: ./features/wallet/driver/transactions/readme.md
```


> **WARNING**
>
> If your README documentation is formatted in DocC, do not add it to the `TOC.yaml` structure until the DocC format becomes available on the Documentary Platform (for the feature implementation ask in the `#documentary` Slack channel).




