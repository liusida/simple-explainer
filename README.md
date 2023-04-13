# simple-explainer
![Simple Explainer](./public/simple-explainer-128.png)
Simple explainer for language learners to understand words or phrases.

![Screenshot](./public/screenshot.png)

## How to use
To use this extension, you can follow these steps:

1. Clone or download the project to your computer.
2. Look for the ./dist folder within the project files, as this folder contains all the necessary files for the extension to work.
3. Open Google Chrome and go to the "Manager Extensions" page. You can access this page by clicking on the three dots at the top right corner of Chrome, then selecting "More Tools" and "Extensions".
4. Once you're on the "Manager Extensions" page, enable the "Developer mode" option located at the top right corner of the page.
5. After enabling the "Developer mode", you should see a new option appear at the top left corner of the page called "Load unpacked". Click on it.
6. Select the ./dist folder from the project files that you've downloaded or cloned earlier.
7. The extension should now be installed and ready to use in Google Chrome.

## Project development setup
```
npm install
```

### Compiles and minifies for production
```
npm run build
```

### Compiles and minifies for development
```
npm run build-watch
```

Then the folder `/dist` contains the extension.

## Acknowledge
The template of this project was created via
```
vue create simple-explainer
cd simple-explainer
vue add chrome-extension-cli
```

Project icon was made via MidJourney
