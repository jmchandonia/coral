# CORAL UI

This is the user facing portion of CORAL. It is designed to simplify tasks
on our system that can be challenging for data producers and data consumers alike. 

## Installation

To get started, make sure you have npm and Angular installed.  This
app doesn't build with node.js versions 17 or higher, so it is likely
that the nodejs and npm versions installed on your linux will not
work.  To install node.js version 16, download
https://nodejs.org/dist/latest-v16.x/ (get the file ending in
linux-x64.tar.gz) and install the contents under /usr/local/.

This app is built with Angular version 11.2.9.  To install 11.2.9, do:

`npm install -g @angular/cli@11.2.9`

Fork and clone a repo and then install all the necessary dependencies in package.json. The command `npm ci` is recommended over `npm install`, as `install` will make updates to the package-lock.json file and may cause complications with conflicting files. `npm install` should only be used if a new dependency needs to be added to the repo.

To run a development server, use the `ng serve` command and open your preferred browser to localhost:4200. To use a specific environment (e.g. production, development) with which to serve the app, you can pass the `configuration` flag to the serve command like so:

`ng serve --configuration=development`

This will tell your application to look at your `environment.dev.ts` file for environment variables. You can use the short form `--prod` instead of `--configuration=production` for serving the prod example.

## Environment

Once the program is installed, you will need to configure environment variables for the application. in `src/environments`, create a file called `environment.ts` if one does not already exist. You can create an `environment.prod.ts` and an `environment.dev.ts` to create different environments for development and production. The required environment variables are as follows:

`production` - Boolean indicating whether to build or serve in production mode. Setting it to false will allow the developer to more easily navigate certain parts of CORAL UI without validation.

`baseURL` - The URL location of your backend API

`GOOGLE_MAPS_API_KEY` - Required for plotting maps of items with geographic data. You will need to create a Google Maps API key using the Google developer console. [Click here](https://developers.google.com/maps/documentation/embed/get-api-key) for more information on how to generate a key. If you do not have access to a Google Maps API key, you can simply leave the GOOGLE_MAPS_API_KEY as an empty string `''` and map options will not be loaded as a part of the UI.

`GOOGLE_OAUTH2_CLIENT_KEY` - Required for authentication - The token provided by the google developer API that allows you to use their OAuth2 services. See the back_end folder's README for more detalis on configuring authentication across the application; it requires steps within the front and the back end.

`GOOGLE_CAPTCHA_SITE_KEY` - Required for setting up user registration. Generates a captcha for preventing site abuse. This option is not required, if you prefer to not use the user registration feature, you must leave this option as an empty string `''`. This will simply prevent the captcha from loading, so if you do have the backend configured with an email for user registration, it would be wise to deconfigure it.

`PROJECT_NAME` - Required in order to display your project name on the first screen after logging in.

`DEMO_MODE` - If set to true, allows ANY user with a Google account to log in and explore data.  The registration button will not be shown.  Uploading new datasets through the web interface will be disabled.  The default is false, meaning authorized users must be configured in users.json (see back end documentation for details).

Please note that without these variables declared as keys within your `environment.*.ts` file(s), the application will fail to compile. The `environment.ts` that lives in the repo is built so that the app won't fail at the compilation step, but will still require the fields to be populated with a valid value, notably the `baseURL` and the `GOOGLE_OAUTH2_CLIENT_KEY` variables.


## Deployment

Note: these directions assume building the site in an Apache2 environment.

After installing the back end, copy all files from this front_end repository into `/home/coral/env/coral-ui/`:

```
cp -r . /home/coral/env/coral-ui
cd /home/coral/env/coral-ui
```


Make sure that mod_rewrite is enabled and add the following to your httpd.conf file: 

```
<Directory "/var/www/html/coral-ui">
    RewriteEngine On
    RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} -f [OR]
    RewriteCond %{DOCUMENT_ROOT}%{REQUEST_URI} -d
    RewriteRule ^ - [L]

    RewriteRule ^ index.html
</Directory>
```
Since CORAL UI is a single page application, these rules are important because they tell the server to redirect any requests that should be handled on the front end back to the index.html page which they reside. Without these rules, the app can have strange behavior such as giving a 404 on refreshing or page navigation.

It is also possible to add these conditions in a .htaccess file at the same level as the app target directory, although the prior method is recommended. If you prefer using .htaccess, you will need to configure a new .htaccess file with the same contents for every new build.

You also need to be sure .htaccess files are enabled in your web server setup; in Apache, be sure "AllowOverride All" is configured for the build target directory.

To install, go to the installation directory at `/home/coral/env/coral-ui/`.
Make any desired changes and then run the following build command as root:

`npm install`

`ng build --prod --base-href /coral-ui/`

The build target directory is configured in angular.json to default to `/var/www/html/coral-ui/`. In order for the resources to be loaded correctly, the base href needs to be set to `/coral-ui/`. This can either be done with the `--base-href` flag as demonstrated above or you can manually change it in the index.html generated by the build command.

Note that everything in the build target directory will be deleted!

The --prod directive above configures the UI with the "prod"
environment, which is configured via src/environments/environment.prod.ts

## Testing

To manually run a test, you can activate the test script using `ng test`. In your terminal, the app will display building information and then show a timestamp of when it was connected to the server. If you are running on your local machine, the default launcher will be with Chrome. You can configure this option in `karma.conf.js`.

On your local machine, the browser will open a GUI that will display the test results. you can refer to the [karma docs](https://karma-runner.github.io/latest/index.html) for more information.

To test in a command-line only environment, e.g. a CI server, you can run the test with the same command while adding the following flags:

`ng test --browsers=ChromeHeadless --watch=false`

Specifying 'ChromeHeadless' will launch a headless instance of a chromium browser with which to run the tests. Make sure you have chromium installed the server. If you get an error from Karma saying that there is no binary for ChromeHeadless browser, you will need to configure an environment variable pointing to the location of your installed chromium:

`export CHROME_BIN=/usr/bin/chromium-browser`

the `--watch=false` flag is not necessary for single runs, but is important for running automated testing as it will terminate once the tests have been run, rather than wait for changes as is the default behavior.

**Important Note**: If you run into an issue where the test server disconnects, you can troubleshoot by inspecting the karma page and viewing the javascript console. There are occasionally errors that will not display in the command line that will log to the browser console.

## Overview

coral-ui is structured into modules that pertain to the part of the site the user is visiting. The following diagram is a high
level overview of the structure of the app.

```
app.module
|__ search.module
|   |__ search components
|__ upload.module
|   |__ upload components
|__ plot.module
|    |__ plot components
|__ shared folder
    |__ components
    |__ services
    |__ directives
    |__ models
    |__ pipes

```

 - components that do not require services (a simple display page) are declared at the app.module level and can be found in
`/src/app/shared/components`. 

 - each child module has 1 or more service(s) that act as controllers for the components within the modules. All service files 
 can be found in `/src/app/shared/services/`.

 - to encourage type safety, patterns that are commonly used throughout the app are stored as ES6 classes and can be found in
 `/src/app/shared/models`.

## Dependencies

 The following is a list of some of the core UI dependencies that our app depends on. It is not a comprehensive list but provides the most widely used 3rd party modules throughout the app.

 **PlotlyJS**

 [PlotlyJS](https://plot.ly/javascript/) is used to translate data bricks into data visualizations that can be configured and viewed by
 the user. it is configured to work with angular using [angular-plotly](https://github.com/plotly/angular-plotly.js).

 Angular-plotly provides us with a simple to use `<plotly-plot>` component that takes a `data` and `layout` input. Server calls typically provide data and layout in separate objects in the response.

 *Important:* We are currently running angular-plotly at version 1.3.2 to prevent an issue that breaks
 changes for angular versions below 8. see [here](https://github.com/plotly/angular-plotly.js/issues/79) for more details.

 **jQuery**
 
[jQuery](https://jquery.com/) is installed in order to support custom UI element plugins, namely DataTables.

**DataTables**

[DataTables](https://datatables.net/) is used for rendering HTML tables with javascript to add search, filtering, and pagination to large tables. DataTables is currently implemented in coral-ui without an 
angular wrapper, so `$` will need to be imported from jQuery in order to render a table as a DataTable.
It is best to render a table in the AfterViewInit lifecycle hook with an ElementRef provided by angular 
as shown below:

```javascript
import { ElementRef, ViewChild } from '@angular/core';
import * as $ from 'jquery';
import 'datatables.net';
import 'datatables.net-bs4'; // for bootstrap 4 styles
...
export class SomeComponent implements afterViewInit {
    @ViewChild('table', { static: false }) private el: ElementRef;
    dataTable: any;

    ngAfterViewInit() {
        this.someService.getSomeData().subscribe(data => {
            // assign data to table
            const table: any = $(this.el.nativeElement);
            this.dataTable = table.DataTable();
        });
    }
}
```

**NgSelect**

[NgSelect](https://ng-select.github.io/ng-select#/data-sources) is used to create comboboxes that populate with data from our system. NgSelect elements can be rendered using a `<ng-select>` tag and take an input of `items`, where items is the data with will populate the dropdown and options take the configuration for the dropdown. 

The text value that the user will see can be configured via the `bindLabel` property, which can be any of the values of the object that you're feeding the data to. 

You can also specify which property you want to model the data with via the `bindValue` property. The default value is the object selected in the `[items]` array. You can access user input selection via the `(change)` event. You can select a default value for an item with the `[(ngModel)]` directive.

component.ts file:
```javascript
    data = [
        {
            firstName: 'Elton',
            lastName: 'John'
        },
        {
            firstName: 'Bob',
            lastName: 'Marley'
        }
    ]

    handleChange(event) {
        console.log(event);
    }
```

component.html file:
```html
    <!-- example without bindValue (logs {firstName: '...', lastName: '...'}) -->
    <ng-select
        [items]="data"
        bindLabel="firstName"
        (change)="handleChange($event)"
    ></ng-select>

    <!-- example with bindValue (logs lastName) -->
    <ng-select
        [items]="data"
        bindLabel="firstName"
        bindValue="lastName"
        (change)="handleChange($event)"
    ></ng-select>
```
**Bootstrap 4 and Ngx-Bootstrap**

for CSS UI, [Bootstrap 4](https://getbootstrap.com/docs/4.0/getting-started/introduction/) is used along 
with [ngx-bootstrap](https://valor-software.com/ngx-bootstrap/#/) for advanced javascript features (e.g
modals and tooltips).
