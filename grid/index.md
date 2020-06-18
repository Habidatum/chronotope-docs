## Basic concepts:

Each data layer shows spatial-temporal aggregated values by each geometry and selected slice duration. For instance, hourly (`PT1H`), daily (`P1D`), weekly (`P1W`) are a slice durations. They have a [special iso format](https://en.wikipedia.org/wiki/ISO_8601).
Each cell on the map - is object, which is built by [quad tile index algorithm](https://wiki.openstreetmap.org/wiki/QuadTiles). We can set different `zoom level` parameter for spatial cell size. `zoom level` value can be between 1 and 26.

Sometimes you need to get average values of slices between specific time intervals. To do this you need set `requestType` parameter for endpoints with data retrieving. This parameter has a two available values: `slice` and `range`. It should be noted that, depending on this parameter, the names of the properties of each cell will differ. For instance, if you request data with `range` request type you get a key `range` in properties by each geometry. But for `slice` request type you get multiple keys in properties, data by each slice in ISO datetime format, for example, `2020-01-01T00:00:00`.

## Layers and their properties

Each layer has a particular properties of data. Before data accessing you should retrieve layer meta information.
There are the following properties:

- `id` - it's a unique identifire of layers
- `label` and `description` - human-readable textual information about layer
- `provider` - data provider company
- `unit` - unit of data in the layer
- `availableSliceDurations` - available slice durations
- `minZoomLevel` and `maxZoomLevel` - min and max zooms of spatial aggregation
- `availableTimeIntervals` - available time slots when there is data
- `initialState` - initial state of layer. It's needed for layer initialization on the map. It has the following properties:
  - `lat` and `lon` - center of viewport on the map
  - `zoomLevel` - initially zoom for spatial data aggregation
  - `sliceDuration` - default selected slice duration stands for temporal data aggregation
  - `timeInterval` - initially selected time intervals for which data is requested

## Access

You need to provide for each endpoint request additional get parameter `access_key`

## Provided API endpoints:

- `/layers` - returns all available layers for you by given token

  ## Example:

  ```shell
  curl -X GET http://api_host/layers
  ```

  ### Response

  ```json
  [
    {
      "id": "layer_id_one",
      "label": "Name of layer",
      "description": "description",
      "provider": "Data provider company",
      "unit": "Users"
    },
    {
      "id": "layer_id_two",
      "label": "Name of layer",
      "description": "description",
      "provider": "Data provider company",
      "unit": "Users"
    }
  ]
  ```

- `/layers/:id` - returns the information by given layer id or send 404 error otherwise

  ## Example:

  ```shell
  curl -X GET http://api_host/layers/layer_id_one
  ```

  ### Response

  ```json
  {
    "id": "layer_id_two",
    "label": "Name of layer",
    "description": "Description of layer",
    "provider": "Data provider company",
    "unit": "Users",
    "maxZoomLevel": 19,
    "minZoomLevel": 10,
    "initialState": {
      "lat": 43.249464,
      "lon": 76.941858,
      "zoomLevel": 16,
      "sliceDuration": "P1D",
      "timeInterval": "2020-03-25T12:00:00/2020-03-30T16:00:00"
    },
    "availableSliceDurations": ["PT1H", "P1D", "P1W"],
    "availableTimeIntervals": [
      "2015-01-01T00:00:00/2017-01-01T00:00:00",
      "2018-06-01T00:00:00/2020-01-01T00:00:00"
    ]
  }
  ```

- `/data` - returns geojson data for given layer and spatial-temporal parameters.

  ## Required parameters:

  - `layerId`
  - Spatial restrictions - viewport parameters: `leftLon`, `rightLon`, `bottomLat`, `topLat`
  - Temporal restrictions - time intervals parameters: `startDate`, `endDate`
  - Spatial-temporal cell parameters: `zoom`, `sliceDuration`
  - Request Type parameter: `requestType`. It has a two available values - `range` and `slice`
  - Response type format `responseTypeFormat` - you can provide desired format for http response. There are multiply available response formats: `csv`, `geojson`, `mvt`. Notice that in `csv` type format you get a data without geometry. By default endpoint provide data in `geojson` type format.

  ## Examples:

  ### Slice Request

  ```shell
  curl -X POST http://api_host/data -d '{
    "layerId": "layer_id_one",
    "leftLon":-0.3087,
    "rightLon":0.1458,
    "bottomLat":51.4156,
    "topLat":51.5533,
    "startDate":"2018-04-15T00:00:00",
    "endDate":"2018-04-20T00:00:00",
    "zoom":16,
    "sliceDuration":"P1D",
    "requestType": "slice"
  }
  '
  ```

  ### Slice Response

  ```json
  {
  "data": [
    {
      "geometry": {
        "coordinates": [
          [
            [0.181274414037248, 51.3957783874423],
            [0.186767578098983, 51.3957783874423],
            [0.186767578098983, 51.3923508697617],
            [0.181274414037248, 51.3923508697617],
            [0.181274414037248, 51.3957783874423]
          ]
        ],
        "type": "Polygon"
      },
      "id": 2552042722689024,
      "properties": {
        "2018-04-17T00:00:00": 44,
        "2018-04-18T00:00:00": 51,
        "2018-04-19T00:00:00": 110,
        "2018-04-20T00:00:00": 27
      },
      "type": "Feature"
    },
    ...
  ]
  }
  ```

  ### Range Request

  ```shell
  curl -X POST http://api_host/data -d '{
    "layerId": "layer_id_one",
    "leftLon":-0.3087,
    "rightLon":0.1458,
    "bottomLat":51.4156,
    "topLat":51.5533,
    "startDate":"2018-04-15T00:00:00",
    "endDate":"2018-04-20T00:00:00",
    "zoom":16,
    "sliceDuration":"P1D",
    "requestType": "range"
  }
  '
  ```

  ### Range Response

  ```json
  {
  "data": [
    {
      "geometry": {
        "coordinates": [
          [
            [0.181274414037248, 51.3957783874423],
            [0.186767578098983, 51.3957783874423],
            [0.186767578098983, 51.3923508697617],
            [0.181274414037248, 51.3923508697617],
            [0.181274414037248, 51.3957783874423]
          ]
        ],
        "type": "Polygon"
      },
      "id": 2552042722689024,
      "properties": {
        "range": 58
      },
      "type": "Feature"
    },
    ...
  ]
  }
  ```

- `data/:tile_zoom/:tile_x/:tile_y` - returns geojson data for given tile, layer and spatial-temporal parameters. If you want to use tile specification you can use this endpoint.

  ## Required parameters:

  - `layerId`
  - Spatial restrictions in GET parameters - : `tile_x`, `tile_y`, `tile_zoom` - tile coordinates
  - Temporal restrictions - time intervals parameters: `startDate`, `endDate`
  - Spatial-temporal cell parameters: `zoom`, `sliceDuration`
  - Response type format `responseTypeFormat` - you can provide desired format for http response. There are multiply available response formats: `csv`, `geojson`, `mvt`. Notice that in `csv` type format you get a data without geometry. By default endpoint provide data in `geojson` type format.

  ## Example

  ```shell
  curl -X POST http://api_host/data/11/1025/682 -d '{
    "layerId": "layer_id_one",
    "startDate":"2018-04-15T00:00:00",
    "endDate":"2018-04-20T00:00:00",
    "zoom":16,
    "sliceDuration":"P1D",
    "requestType": "slice"
  }
  '
  ```

  ## Response

  ```json
  {
    "data": [
      {
        "geometry": {
          "coordinates": [
            [
              [0.181274414037248, 51.3957783874423],
              [0.186767578098983, 51.3957783874423],
              [0.186767578098983, 51.3923508697617],
              [0.181274414037248, 51.3923508697617],
              [0.181274414037248, 51.3957783874423]
            ]
          ],
          "type": "Polygon"
        },
        "id": 2552042722689024,
        "properties": {
          "2018-04-17T00:00:00": 44,
          "2018-04-18T00:00:00": 51,
          "2018-04-19T00:00:00": 110,
          "2018-04-20T00:00:00": 27
        },
        "type": "Feature"
      },
      ...
    ]
  }
  ```
