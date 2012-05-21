jQuery filedrop plugin - html5 drag desktop files into browser
==============================
Forked from: https://github.com/weixiyen/jquery-filedrop

jQuery filedrop uses the HTML5 File API to allow users
to drag multiple files from desktop to the browser, uploading
each file to a user-specified URL.

filedrop uses HTML5 FileReader() to read file data.

Browser Support
---------------
Works on Chrome and Firefox 3.6+.

Safari has issues due to its lack of Filereader support which is required in order to
build the multi part request body in the format that S3 requires.

filedrop also allows users to define functions to handle the 'BrowserNotSupported' error.

Usage Example
---------------

    $('#dropzone').filedrop({
        fallback_id: false,
        queuefiles: 2,
        maxfiles: 20,
        maxfilesize: 20,
        url: window.app.s3_proxy_url,
        paramname: 'file',
        data: {
          Filename: function(filename) {
            return filename;
          },
          success_action_status: "201",
          acl: function(filename){
            return window.my_upload_handler.s3_request_params[filename].acl;
          },
          key: function(filename) {
            return window.my_upload_handler.s3_request_params[filename].key;
          },
          'Content-Type': function(filename, file_type) {
            return file_type;
          },
          signature: function(filename) {
            return window.my_upload_handler.s3_request_params[filename].signature;
          },
          'Content-Disposition': function(filename) {
            return window.my_upload_handler.s3_request_params[filename].content_disposition;
          },
          AWSAccessKeyId: function() {
            return window.app.s3_access_key;
          },
          policy: function(filename) {
            return window.my_upload_handler.s3_request_params[filename].policy;
          }
        },
        error: function(err, file) {
          switch(err) {
            case 'BrowserNotSupported':
              console.log('Your Browser does not support html5 drag and drop')
              break;
            case 'TooManyFiles':
              alert("Too many files were selected");
              break;
            case 'FileTooLarge':
              alert("One or more files exceed the maximum individual file size for drag and drop, of up to " + self.max_file_size + "MB, To upload large files please use the upload button.");
              break;
            default:
              alert("An unknown error occurred while attempting to perform your upload, if this continues please use the upload button instead.")
              break;
          }
        },
        dragOver: function(e) { },
        dragLeave: function(e) { },
        docOver: function(e) { },
        docLeave: function(e) { },
        drop: function(files, e) { },
        uploadStarted: function(i, file, len, xhr_request){ },
        uploadFinished: function(i, file, response, time) { },
        progressUpdated: function(i, file, progress) { },
        speedUpdated: function(i, file, speed){},
        rename: function(name) {
          return name;
        },
        beforeEach: function(file, callback) {
          // this method will need to run callback() after fetching s3 params
          // in order to trigger the upload
          window.my_upload_handler.get_s3_request_params(file, callback);
        },
        afterAll: function(){}
      });


Queueing Usage Example
----------------------

To enable the upload of a large number of files, a queueing option was added that enables you to configure how many files should be processed at a time.  The upload will process that number in parallel, backing off and then processing the remaining ones in the queue as empty upload slots become available.

This is controlled via one of two parameters:

    maxfiles: 10    // Limit the total number of uploads possible - default behaviour
    queuefiles: 2   // Control how many uploads are attempted in parallel (ignores maxfiles setting)

Not setting a value for queuefiles will disable queueing.

Gotchas
-------

In order to build the upload request we use Filereader to read binary objects into memory and encode for uploading.
As a result this affects large file uploads which become memory intensive for the client.
Even though S3 allows up to 5GB per object, take heed when setting the
maxfilesize and queuefiles (number of simultaneous uploads) param.
Filereader usage also means Safari and IE are currently unsupported.


Contributions
---------------
[weixiyen](https://github.com/weixiyen/jquery-filedrop) (Weixi Yen)
[Reactor5](http://github.com/Reactor5/) (Brian Hicks)
[jpb0104](http://github.com/jpb0104)
