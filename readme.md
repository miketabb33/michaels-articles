# Michael's Articles

The data source for the articles in [Michael's Website](https://github.com/miketabb33/michaels-page).

## How To Add An Article To The Website
1. Add the new article as a readme file under the articles directory.

2. Then add an entry to the `manifest.json` file. Follow formatting seen in other `manifest.json` entries.

3. Hide work in progress articles in the production environment by adding the following optional property to the `manifest.json` entry: `hidInProd: true`

Example:
``` json
  {
    "path": "any-path",
    "slug": "any-slug",
    "title": "any-title”",
    "description": "any-description",
    "thumbnailUrl": "https://i.ibb.co/rMh2Rp1/oauth-image.jpg",
    "createdOn": "2024-03-22T23:03:41.000Z",
    "hideInProd": true <--- HERE
  },
```
