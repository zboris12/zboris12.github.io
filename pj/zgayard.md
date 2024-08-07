---
title: zgayard | Create a safety yard in your online storage
description: A security tool to encrypt or decrypt files in Microsoft Onedrive or Google Drive.
last_modified_at: 2024-03-25T18:21:44+09:00
---
<div align="center"><img src="https://zboris12.github.io/zgayard/src/img/logo.png" title="zgayard"></div>

# [zgayard](https://github.com/zboris12/zgayard)
![version](https://img.shields.io/github/package-json/v/zboris12/zgayard)
![license](https://img.shields.io/github/license/zboris12/zgayard)
![build status](https://img.shields.io/badge/build-passing-brightgreen?logo=cloudflare)

Create a safety yard in your online storage. [Let's go!](https://zgayard.pages.dev/)  

This is a web tool aimed to make your online storage more safety.  
It is a single page application and almost all processing is performed in the web browser.  
The server side is only used to handle authorization of online storage, which can't be processed in the web browser.

PS: __ZGA__ is the abbreviation of my father's name.  
And I use this name to hope the merits from this application will be dedicated to my parents.

## NOTE: it is still in development

* Only support a few online storages now.

## Main features

* Manage your online storage. Include:
  * Create a new folder
  * Upload files
  * Download files
  * Rename files or folders
  * Move files or folders
  * Delete files or folders
* Encrypt file when upload to online storage. Also encrypt file's name if the setting is chosen.
* Decrypt file when download from online storage.
* Support multiple root directories with different password and key files.
* Support to upload folder.
* Support to view image file or video file directly from online storage.
* Automatically play the next video file after the current video is ended.
* Supported online storage:
  * Microsoft Onedrive
  * Google Drive

## TODO

* Support to download multiple files and compress them to a zip file
* More online storage support
* More language support
* Support to change key and password

## The Dependencies

* ~~[crypto-js](https://github.com/brix/crypto-js)~~
* [node-forge](https://github.com/digitalbazaar/forge)  
Changed crypto module from crypto-js to node-forge, because node-forge is fast.
* ~~[videostream](https://github.com/jhiesey/videostream)~~  
Because Safari on iOS does not support [MediaSource](https://developer.mozilla.org/ja/docs/Web/API/MediaSource),
the new version uses service worker to handle video requests instead.

## The Icons

The svg icons are got from [iconsvg](https://iconsvg.xyz/).

## The UI

The UI of the webpage is powered by [Tailwind CSS](https://tailwindcss.com/).

## License

This application is available under the
[MIT license](https://opensource.org/licenses/MIT).  

[🚗BACK](/README.html)
