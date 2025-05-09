# Dynamic Web TWAIN Vue Advanced

This project was bootstrapped with [Create Vue App](https://github.com/vuejs/vue-cli). It utilizes the SDK [Dynamic Web TWAIN](https://www.dynamsoft.com/Products/WebTWAIN_Overview.aspx) to provide the following functionalities

* Scan documents from physical scanners
* Capture images from webcams
* Load documents from local disk (bmp/jpg/png/tif/pdf)
* View and process the documents already scanned/captured/loaded
  * Editing (rotate/flip/mirror/crop, etc.)
  * Saving or uploading
  * Barcode reading

## Usage

Environment: Node.js v22.14.0

1. Apply for a [30-day free trial license](https://www.dynamsoft.com/customer/license/trialLicense?product=dwt) of Dynamic Web TWAIN.

2. Update the license key in `src\components\DynamsoftSDK.vue` file:

   ```
   Dynamsoft.DWT.ProductKey = "LICENSE-KEY";
   ```

3. Install the dependencies:

   ```
   yarn install
   ```

4. Run the application as follows:

   ```
   yarn serve
   ```

5. Set up and run the server piece which is used solely for receiving uploaded files. Run the  command-line as follows:

   ```
   cd server
   yarn install
   yarn start
   ```

## Build

Run `yarn build` to build the project. The build artifacts will be stored in the `build/` directory. 
It correctly bundles Vue in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.<br />
Your app is ready to be deployed!

