# Pain Management Summary SMART on FHIR Application

## About

The Pain Management Summary SMART on FHIR application was developed to support the pilot of the CDS artifact, [Factors to Consider in Managing Chronic Pain: A Pain Management Summary](https://cds.ahrq.gov/cdsconnect/artifact/factors-consider-managing-chronic-pain-pain-management-summary).  This artifact presents a variety of key "factors" for clinicians to consider when assessing the history of a patient's chronic pain.  These factors include subjective and objective findings, along with recorded treatments and interventions to inform shared decision making on treatments moving forward.

The Pain Management Summary SMART on FHIR application was piloted during Summer 2018.  Local modifications and development were needed to fully support this application in the pilot environment.  For example, custom development was needed to expose pain assessments via the FHIR API. See the pilot reports for more information.

This application was originally piloted with support for FHIR DSTU2.  The app has been updated since the pilot to also support FHIR R4, although R4 support has not been piloted in a clinical setting.  In addition, other changes in logic and terminology have been made via annual updates. For more information, see the CQL change log attached to the [Factors to Consider in Managing Chronic Pain: A Pain Management Summary](https://cds.ahrq.gov/cdsconnect/artifact/factors-consider-managing-chronic-pain-pain-management-summary) artifact on CDS Connect.

Taking steps to ensure accessibility by the widest range of users, an accessibility subject matter expert performed a review of the application, enumerated issues found, and provided recommended remediations.  In addition to the recommendations, the [Mozilla ARIA Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA) reference was used to address issues.  The application was then manually tested using accessibility tools including JAWS, VoiceOver, and the [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/).

This prototype application is part of the [CDS Connect](https://cds.ahrq.gov/cdsconnect) project, sponsored by the [Agency for Healthcare Research and Quality](https://www.ahrq.gov/) (AHRQ), and initially developed under contract with AHRQ by MITRE's [Health FFRDC](https://www.mitre.org/our-impact/rd-centers/health-ffrdc).

## Contributions

For information about contributing to this project, please see [CONTRIBUTING](CONTRIBUTING.md).

## Development Details

The Pain Management Summary is a web-based application implemented with the popular [React](https://reactjs.org/) JavaScript framework. The application adheres to the [SMART on FHIR](https://smarthealthit.org/) standard, allowing it to be integrated into EHR products that support the SMART on FHIR platform. To ensure the best adherence to the standard, the Pain Management Summary application uses the open source [FHIR client](https://github.com/smart-on-fhir/client-js) library provided by the SMART Health IT group.

The logic used to determine what data to display in the Pain Management Summary is defined using [CQL](http://cql.hl7.org/) and integrated into the application as the corresponding JSON ELM representation of the CQL.  The application analyzes the JSON ELM representation to determine what data is needed and then makes the corresponding queries to the FHIR server.

Once the necessary FHIR data has been retrieved from the EHR, the open source [CQL execution engine](https://github.com/cqframework/cql-execution) library is invoked with it and the JSON ELM to calculate the structured summary of the data to display to the user.  This structured summary is then used by the React components to render a user-friendly view of the information.

### Limitations

This CDS logic queries for several concepts that do not yet have standardized codes.  To support this, the following local codes have been defined:

| Code | System | Display |
| --- | --- | --- |
| SQETOHUSE | http://cds.ahrq.gov/cdsconnect/pms | Single question r/t ETOH use |
| SQDRUGUSE | http://cds.ahrq.gov/cdsconnect/pms | Single question r/t drug use |
| MME | http://cds.ahrq.gov/cdsconnect/pms | Morphine Milligram Equivalent (MME) |

Systems integrating the Pain Management Summary will need to expose the corresponding data as observations using the codes above.  As standardized codes become available, these local codes will be replaced.

### To build and run in development:

1. Install [Node.js](https://nodejs.org/en/download/) (LTS edition, currently 16.x)
2. Install dependencies by executing `npm install` from the project's root directory
3. If you have a SMART-on-FHIR client ID, edit `public/launch-context.json` to specify it
4. NOTE: The launch context contains `"completeInTarget": true`. This is needed if you are running in an environment that initializes the app in a separate window (such as the public SMART sandbox).  It can be safely removed in other cases.
5. If you'll be launching the app from an Epic EHR, modify `.env` to set `VITE_EPIC_SUPPORTED_QUERIES` to `true`
6. Serve the code by executing `npm start` (runs on port 8000)

### To build and deploy using a standard web server (static HTML and JS)

The Pain Management Summary can be deployed as static web resources on any HTTP server.  There are several customizations, however, that need to be made based on the site where it is deployed.

1. Install [Node.js](https://nodejs.org/en/download/) (LTS edition, currently 16.x)
2. Install dependencies by executing `npm install` from the project's root directory
3. Modify the `base` value in `vite.config.mjs` to reflect the path (after the hostname) at which it will be deployed
   a. The path must start and end with a forward slash (`/`).
   b. For example, if deploying to https://my-server/pain-mgmt-summary/, the `base` value should be `"/pain-mgmt-summary/"`.
   c. If deploying to the root of the domain, set the `base` value to `"/"` or comment out the `base` property.
4. Modify the `clientId` in `public/launch-context.json` to match the unique client ID you registered with the EHR from which this app will be launched
5. NOTE: The launch context contains `"completeInTarget": true`. This is needed if you are running in an environment that initializes the app in a separate window (such as the public SMART sandbox).  It can be safely removed in other cases.
6. If you've set up an analytics endpoint (see below), set the `analytics_endpoint` and `x_api_key` in `public/config.json`
7. If you'll be launching the app from an Epic EHR, modify `.env` to set `VITE_EPIC_SUPPORTED_QUERIES` to `true`
   a. This modifies some queries based on Epic-specific requirements
8. Run `npm run build` to compile the code to static files in the `dist` folder
9. Deploy the output from the `dist` folder to a standard web server

Optionally to step 9, you can run `npm run serve` to use Vite's built-in server to host the code in `dist`. This approach, however, should not be used in production.

### To update the CQL and/or ELM JSON files

The CQL source and ELM JSON files are kept in the `src/cql/dstu2` and `src/cql/r4` folders. The Pain Management Summary app executes only the ELM JSON files, but it is helpful to keep the CQL source files alongside them.

To update the CQL files:

1. Download the CQL zips from the [CDS Connect artifact](https://cds.ahrq.gov/cdsconnect/artifact/factors-consider-managing-chronic-pain-pain-management-summary) or get them from another source (if applicable).
2. Unzip the FHIR DSTU2 / 1.0.2 CQL package and copy the CQL and JSON files to `src/cql/dstu2`
3. Unzip the FHIR R4 / 4.0.1 CQL package and copy the CQL and JSON files to `src/cql/r4`

The ELM JSON files in the distributable zip packages include optional annotations, locators, result types, and signatures. These are not required by this application and significantly increase its file size. The ELM JSON files should be rebuilt without this optional data in order to improve the efficiency of this app.

To rebuild the ELM JSON files without annotations, locators, result types, and unnecessary signatures:

1. Install [JDK 11](https://adoptium.net/temurin/releases/?version=11) or greater (if necessary)
2. Note the `translatorVersion` in one of the ELM JSON files from the downloaded zip package (e.g., `3.10.0`)
3. Update the `runtimeOnly` dependency's version in the `build.gradle` file (e.g., `runtimeOnly 'info.cqframework:cql-to-elm-cli:3.10.0'`)
4. Run `npm run cql-to-elm` to rebuild the ELM JSON files (note: warnings are expected, but there should be no errors)
5. Review the changes in all ELM JSON files in the `src/cql` folders to ensure they look correct

### To update the valueset-db.json file

The value set content used by the CQL is cached in a file named `valueset-db.json`.  If the CQL has been modified to add or remove value sets, or if the value sets themselves have been updated, you may wish to update the valueset-db.json with the latest codes.  To do this, you will need a [UMLS Terminology Services account](https://uts.nlm.nih.gov//license.html).

To update the `valueset-db.json` file:

1. Run `node src/utils/updateValueSetDB.js UMLS_API_KEY` _(replacing UMLS\_API\_KEY with your actual UMLS API key)_

To get you UMLS API Key:

1. Sign into your UMLS account at [https://uts.nlm.nih.gov/uts.html](https://uts.nlm.nih.gov/uts.html)
2. Click 'My Profile' in the orange banner at the top of the screen
3. Your API key should be listed below your username in the table
4. If no API key is listed:
   1. Click ‘Edit Profile’
   2. Select the ‘Generate new API Key’ checkbox
   3. Click ‘Save Profile’
   4. Your new API key should now be listed.

### To run the unit tests

To execute the unit tests:

1. Run `npm test`

## To test the app using the public SMART sandbox

Run the app via one of the options above, then:

1. Browse to http://launch.smarthealthit.org/
2. Select `R2 (DSTU2)` or `R4` from the FHIR Version dropdown
3. In the _App Launch URL_ box at the bottom of the page, enter: `http://localhost:8000/AHRQ-CDS-Connect-PAIN-MANAGEMENT-SUMMARY/launch.html`
4. Click _Launch App!_
5. Select a patient

### To update the test patients' date-based fields

Testing this SMART App is more meaningful when we can supply test patients that exercise various aspects of the application.  Test patients are represented as FHIR bundles at `src/utils/dstu2_test_patients` and `r4_test_patients`.  Since the CDS uses lookbacks (for example, only show MME in the last 6 months), the patient data occasionally needs to be updated to fit within the lookback windows. To automatically update the data to fit within the lookback windows as of today's date:

1. Run `npm run update-test-patients`

This will update all of the entries in the patient bundles to be appropriate relative to today's date. In addition, it sets each bundle's `meta.lastUpdated` to the current date. This is essential for ensuring that future updates work correctly since it uses the `meta.lastUpdated` date to determine how far back each other date should be relative to today.

### To upload test patients to the public SMART sandbox

Testing this SMART App is more meaningful when we can supply test patients that exercise various aspects of the application.  Test patients are represented as FHIR bundles at `src/utils/dstu2_test_patients` and `r4_test_patients`.  To upload the test patients to the public SMART sandbox:

1. Run `npm run upload-test-patients`

This adds a number of patients, mostly with the last name "Jackson" (for example, "Fuller Jackson" has entries in every section of the app).  The SMART sandbox may be reset at any time, so you may need to run this command again if the database has been reset.

### To test the app in standalone mode using the public SMART sandbox

To enable launching the app via a direct URL (without the SMART Launcher page), you can reconfigure the app as a standalone app.  To do so, follow these steps:

1. Overwrite the `/public/launch-context.json` file with these contents:
   ```json
   {
     "clientId": "6c12dff4-24e7-4475-a742-b08972c4ea27",
     "scope":  "patient/*.read launch/patient",
     "iss": "url-goes-here"
   }
   ```
2. Restart the application server
3. Browse to http://launch.smarthealthit.org/
4. Select `R2 (DSTU2)` or `R4` from the FHIR Version dropdown
5. In _Launch Type_, choose **Provider Standalone Launch**
6. Copy the FHIR URL in the _FHIR Server URL_ box at the bottom of the page (e.g., `http://launch.smarthealthit.org/v/r2/sim/eyJoIjoiMSIsImkiOiIxIiwiaiI6IjEifQ/fhir`)
7. Paste it into `/public/launch-context.json` file where `url-goes-here` is
8. Browse to http://localhost:8000/AHRQ-CDS-Connect-PAIN-MANAGEMENT-SUMMARY/launch.html

_NOTE: Do *not* check in the modified launch-context.json!_

## To test the app using the Epic SMART sandbox

The public Epic sandbox does not provide any synthetic patients that exercise the Pain Management Summary logic very well.  For this reason, testing against the public Epic sandbox is generally only useful to prove basic connection capability.

Run the app via one of the options above, then:

1. Browse to https://open.epic.com/Launchpad/Oauth2Sso
2. Select a patient from the dropdown
3. In the _YOUR APP'S LAUNCH URL_ box, enter: `http://localhost:8000/AHRQ-CDS-Connect-PAIN-MANAGEMENT-SUMMARY/launch.html`
4. In the _YOUR APP'S OAUTH2 REDIRECT URL_ box, enter: `http://localhost:8000/AHRQ-CDS-Connect-PAIN-MANAGEMENT-SUMMARY/`
5. Click _Launch App_

## Advanced: To post application analytics

This app can post JSON-formatted analytic data to an endpoint each time the application is invoked.

The data that is posted reports whether or not the patient met the CDS inclusion criteria, lists each section and subsection of the summary (along with the number of entries in each subsection), and provides an overall count of entries.  The basic form of the data is as follows:

```
{
  "meetsInclusionCriteria": <boolean>,
  "sections": [
    {
      "section": <stringName>,
      "subSections": [
        { "subSection": <stringName>, "numEntries": <intCount> },
        ...
      ]
    },
    ...
  ],
  "totalNumEntries": <intCount>
}
```

To enable posting of analytics, configure the `analytics_endpoint` and `x_api_key` in the `public/config.json` file. The default value is an empty string, which will not post any analytics.

## To Deploy to GitHub Pages

This application can be deployed to GitHub Pages. This is the primary mechanism by which it is made available for testing by the SMART App Gallery.

To deploy to GitHub Pages:

1. Ensure that your git `origin` is set to the GitHub repository (you can check using `git remote -v`).
2. Ensure that you are on the branch you wish to deploy and that you do not have local code modifications that should not be deployed.
3. Run `npm run deploy`.

Once deployed, the launch URL for your deployed app will be:
```
https://ahrq-cds.github.io/AHRQ-CDS-Connect-PAIN-MANAGEMENT-SUMMARY/launch.html
```

## LICENSE

Copyright 2018-2024 Agency for Healthcare Research and Quality

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.