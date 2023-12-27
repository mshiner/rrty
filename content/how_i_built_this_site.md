+++
title = "How I built this site"
date = 2023-12-01
draft =  false
+++

## The Google Sheet

Initially I started adding my RRtY rides to a Google Sheet as I wanted a separate record of my RRtYs distinct from my Strava feed.

The sheet made it easy to record and categorise the rides and do some further analysis - see the [stats page](/stats/1) for some results of this. Obvously I could make the data available to others by sharing the sheet, but I wanted a better way to share my rides with the world - hence this site.

## The api

Step 1 was to create an api for data access.

This would require making the contents of the sheet available in some form digestible outside Google workspace. The normal format for this sort of thing is JSON  - therefore a bit of code in Apps Script attached to the sheet would be rquired as Google Sheets won't output JSON automatically. After a few searches I found [this tutorial](https://ravgeetdhillon.medium.com/turn-a-google-sheet-into-a-rest-api-b08f3fd641ad) which builds the following api interface code:

```js
function json(sheetName) {
  const spreadsheet = SpreadsheetApp.getActiveSpreadsheet()
  const sheet = spreadsheet.getSheetByName(sheetName)
  const data = sheet.getDataRange().getValues()
  const jsonData = convertToJson(data)
  return ContentService
        .createTextOutput(JSON.stringify(jsonData))
        .setMimeType(ContentService.MimeType.JSON)
}

function convertToJson(data) {
  const headers = data[0]
  const raw_data = data.slice(1,)
  let json = []
  raw_data.forEach(d => {
      let object = {}
      for (let i = 0; i < headers.length; i++) {
        object[headers[i]] = d[i]
      }
      json.push(object)
  });
  return json
}

function doGet(e) {
  const path = e.parameter.path
  return json(path)
}
```

So now the data was 'available' on the web. Next thing was to publish it.

## Hugo 'dynamic' pages

My tool of choice for building small sites is [Hugo](https://gohugo.io/), which is a static site generator. From source files (HTML templates, theme styling and Markdown) Hugo will build the pages for a site.

Whoa - hang on, though. A __static__ site means that I'd have to add pages manually from source data (Strava) which would defeat the whole object of the Google Sheet. But.... in more recent versions Hugo has introduced the concept of content from data. Could this be the way to do what I wanted - it turns out it can.

More searches, and I found this [article](https://www.thenewdynamic.com/article/toward-using-a-headless-cms-with-hugo-part-2-building-from-remote-api/) which did EXACTLY what I wanted.

I was now able to dynaically generate web content directly from my sheet.

Finally, on the Hugo side I neededa 'theme'. In general I don't use pre packaged themes because I like to know what is going on with my sites. Since this was a hobby project, and I needed something simple I used [Missing CSS Stylesheet](https://missing.style/) which has pretty good default styling.

## Netlify & GitHub

The easiest (best?) way to deploy a Hugo site is using GitHub repo and [Netlify](https://www.netlify.com/). When you push a change to the repo this triggers a build on Netlify and the site is deployed (i.e. the new content is published).

OK, so far so good, but when I add content to the sheet this doesn't push to the repo. I therefore needed a way to trigger a build in some other way - enter [Netlify Build Hooks](https://docs.netlify.com/configure-builds/build-hooks/) - this allows you to send a payload to an endpoint and trigger the site to be rebuilt.

Cool! So I added a little bit of code to the sheet (from [here](https://gist.github.com/jmolivas/bab53777bee19caebb5ca31d1f8b6e11)) which adds a menu option to trigger the build.

```js
# From your Google Spreadsheet, select the menu item
# Tools > Script editor. 
# Copy and paste this code.
# Replace uuid with the build_hooks uuid from your Netlify project.

function onOpen() {
  SpreadsheetApp.getUi()
      .createMenu('Scripts')
      .addItem('Build', 'build')
      .addToUi();
}

function build() {
  UrlFetchApp.fetch('https://api.netlify.com/build_hooks/uuid', {
    'method': 'post',
  })
}
```
## Wrap Up

This was a fun little project that got me back into Hugo after a bit of a break. It took a bit of time to pull the pices together, but I think the result is pretty good.

All of the code is here on the [GitHub repo](https://github.com/mshiner?tab=repositories)