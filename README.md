POC ATTACHMENT ZIP AND UNZIP

First zipping more than one file into a zip file

Code:
Action chain code:

define([
  'vb/action/actionChain',
  'vb/action/actions',
  'vb/action/actionUtils',
], (
  ActionChain,
  Actions,
  ActionUtils
) => {
  'use strict';

  class FilePickerSelectChain_header_save_demo extends ActionChain {
    /**
     * @param {Object} context
     * @param {Object} params
     * @param {object[]} params.files 
     */
    async run(context, { files }) {
      const { $page, $flow, $application, $constants, $variables, $functions } = context;

      // Allowed file types
      const allowedFileTypes = [
  'application/pdf',
  'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
'application/msword',
  'image/png',
  'image/jpeg',
  'text/csv',
  'audio/mpeg',
  'audio/wav',
  'audio/ogg',
  'video/mp4',
  'video/webm',
  'video/ogg',
];

      if (files && files.length > 0) {
        let validFiles = [];
        let invalidFiles = [];

        for (let selectedFile of files) {
          if (selectedFile.size > 10000000) {
            invalidFiles.push({
              file: selectedFile,
              error: 'File size is larger than 10MB!'
            });
            continue;
          }

          if (!allowedFileTypes.includes(selectedFile.type)) {
            invalidFiles.push({
              file: selectedFile,
              error: 'Invalid file format! Please upload a valid file.'
            });
            continue;
          }

          validFiles.push(selectedFile);
          $variables.filevarname=selectedFile.name;
          // $variables.storeUploadedDocVar = selectedFile;
          // $variables.attachmentPost.DOCUMENT_FILE = selectedFile;
          // $variables.attachmentPost.DOCUMENT_NAME = selectedFile.name;
          // $variables.attachmentPost.DOCUMENT_TYPE = selectedFile.type;

          await Actions.fireNotificationEvent(context, {
            summary: `${selectedFile.name} uploaded successfully.`,
            severity: 'confirmation',
          });
        }

        if (invalidFiles.length > 0) {
          for (let invalid of invalidFiles) {
            await Actions.fireNotificationEvent(context, {
              summary: invalid.error,
              severity: 'error',
            });
          }
        }

        if (validFiles.length === 0) {
          await Actions.fireNotificationEvent(context, {
            summary: 'No valid files were uploaded!',
            severity: 'error',
          });
          return;
        }

        // Create ZIP and download
        await new Promise((resolve, reject) => {
          require([
            'https://cdnjs.cloudflare.com/ajax/libs/jszip/3.6.0/jszip.min.js',
            'https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/1.3.8/FileSaver.js'
          ], function (JSZip, FileSaver) {
            const zip = new JSZip();
        
            files.forEach(file => {
              zip.file(file.name, file);
            });
        
            zip.generateAsync({ type: 'blob' })
              .then(blob => {
                // Save to VBCS variable here
                $variables.generatedZipBlob = blob;
                $variables.fileType = blob.type;
                console.log("ZIP Blob created: ", blob);
        
                // Download the zip
                saveAs(blob, 'uploaded_files.zip');
                resolve();
              })
              .catch(error => {
                console.error('Zip creation failed:', error);
                reject(error);
              });
          });
        });

        const fileToBase64 = await $functions.fileToBase64($variables.generatedZipBlob);

        // ---- TODO: Add your code here ---- //
        console.log('filetobase64bit',fileToBase64);

        $variables.fileContent = fileToBase64;

        $variables.fileName = $variables.filevarname.replace(/\.[^/.]+$/, '') + '.zip';

        await Actions.fireNotificationEvent(context, {
          summary: $variables.fileName,
          message: $variables.fileType,
        });
      } else {
        await Actions.fireNotificationEvent(context, {
          summary: 'No file selected!',
          severity: 'error',
        });
      }
    }
  }

  return FilePickerSelectChain_header_save_demo;
});

Working process:
In this part first I need to add the file picker in the fragment and create three variable for the fragment and also enable the required and write back to container and then copy the code for zipping the file and then showing it in the screen 

Sample file is showing in the github.
