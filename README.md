# cordova_photo_assets
Cordova Plugin for Accessing the Photo Assets on iOS and eventually Android

## API

### Simple Example

This example will list all assets for the first asset-collection found.

```coffeescript

document.addEventListener 'deviceready', ->
  PhotoAssets.setOptions currentAssetCollection: "all", ->
    console.log "all (local) photo assets selected, a photoAssetsChanged event will follow shortly"

# called after PhotoAssets options change OR the local OS notifies us that the photos have changed.
document.addEventListener 'photoAssetsChanged', ({details})->
  {collection, offset, limit, assets} = details
  {collectionKey, collectionName, estimatedAssetCount} = collection

  console.log "#{collectionName} assets: #{offset} to #{offset + assets.length - 1}:"
  for asset in assets
    {
      assetKey
      pixelWidth
      pixelHeight
      thumbnailUrl
      mediaType
      creationDate
      modificationDate
    } = asset
    console.log "asset #{offset++}", asset

  if assets.length == limit
    PhotoAssets.setAssetWindow offset + limit, limit

```

### Extended Example

This example shows how to get a list of all available collections, how to follow one specifically, and how to set all custom options.

```coffeescript

document.addEventListener 'deviceready', ->

  PhotoAssets.getAssetCollections (assetCollections)->
    [{
      collectionKey
      collectionName
      estimatedAssetCount
    }] = assetCollections

    PhotoAssets.setOptions
      currentCollectionKey: collectionKey
      thumbnailSize: 270
      limit: 100
      offset: 0
      thumbnailQuality: 95
    , ->
      console.log "selected first asset collection: " + collectionName

document.addEventListener 'photoAssetsChanged', ({details})->
  # ...
```

### PhotoAssets API DETAILS

#### getAssetCollections
```coffeescript
PhotoAssets.getAssetCollections successCallback, errorCallback

successCallback: (arrayOfCollectionInfoObjects) ->
```

On success, the following is invoked:

```coffeescript
  successCallback [
    # one or more collections with the following format:
    {
      collectionKey:        "all"
      collectionName:       "Camera Roll"
      estimatedAssetCount:  2000
    }
  ]
```

#### setOptions
```coffeescript
PhotoAssets.setOptions options, successCallback, errorCallback

successCallback: ->
```

```options``` can be 0, 1 or more of the following. The missing options will not be changed.

* thumbnailSize:        (pixels)
* thumbnailQuality:     (0-100)
* limit:                (int >= 1) number of thumbnails to return starting from the current offset.
* offset:               (int >= 0) current thumbnail offset
* currentCollectionKey: (string) Use "all" for all local images. Otherwise, get collection keys from getAssetCollections


#### getOptions
```coffeescript
PhotoAssets.getOptions successCallback, errorCallback

successCallback: (options) ->
```

Returns the current value for all options as an ```options``` object
on success: sucessCallback options
wheren 'options' is an object with properties for the current value of all options


#### getPhoto

```coffeescript
PhotoAssets.getPhoto options, successCallback, errorCallback

successCallback: ({photoUrl, pixelWidth, pixelHeight, originalPixelWidth, originalPixelHeight}) ->
```

Options:

* assetKey:           (required) the key of the asset you want the photo for
* maxSize:            (pixels)
* temporaryFilename:  If the name is the same as a previous call, the previous image is overwritten. This is handy so you don't end up with lots of temporary files wasting the users's storage.

#### photoAssetsChanged event

```
document.addEventListener 'photoAssetsChanged', ({details})->
  {collection, offset, limit, assets} = details
  {collectionKey, collectionName, estimatedAssetCount} = collection

  {
    assetKey
    thumbnailUrl
    thumbnailPixelWidth
    thumbnailPixelHeight
    originalPixelWidth
    originalPixelHeight
    mediaType
    creationDate
    modificationDate
  } = assets[0]
```

The event object's ```details```:

* collection: information about the current collection in the same format as returned by ```getCollections```
* offset: the current offset for the data-window
* limit: the current size of the data-window
* assets: an array of assets <= limit in length

The asset objects have the following fields:

* assetKey: used in ```PhotoAssets.getPhoto assetKey: 'abc123', ...``` to retrieve high resolution versions of the asset
* thumbnailUrl: the URL of the immediately-available copy of the thumbnail for this asset
* thumbnailPixelWidth, thumbnailPixelHeight: size of the thumbnail image
* originalPixelWidth, originalPixelHeight: size of the original image
* mediaType: ???
* creationDate: string
* modificationDate: string

# Future

```
iOS supports the following additional attributes for some collections:

  startDate
  endDate
  approximateLocation
  localizedLocationNames

And the following additional asset properties:

  location, duration, favorite, hidden
```
