// A UI to interactively calculate grass cover (%) and grazing capacity (ha/LSU),
// inspect individual pixels, chart annual NPP and export results.

// Load scripts containing model tools.
var tools = require('users/liezlvermeuln/Masters:Model');

// The namespace for our application.  All the state is kept in here.
var app = {};

/** Creates the global state variables. */
var finalLon;
var finalLat;
var calcl8;
var calcmodis; 
var startDate = '2019-01-18';
var endDate = '2019-02-18';
var grazeResult = false;
var grassResult = false;
var globalNPPL8;
var globalNPPMODIS;
var setGrassLS = false;
var setGrassMODIS = false;
var setGrazeLS = false;
var setGrazeMODIS = false;

/** Creates the UI panels. */
app.createPanels = function() {
  /* The introduction section. */
  app.intro = {
    panel: ui.Panel([
      ui.Label({
        value: 'Graze Engine',
        style: {fontWeight: 'bold', fontSize: '24px', margin: '10px 5px', color: '#4d4d4d'}
      }),
      ui.Label('This app allows you to calculate grass cover and grazing capacity using Landsat 8 and/or MODIS imagery')
    ])
  };
  
  /* The location selection section. */
  app.location = {
    lon: ui.Textbox('LONGITUDE', 18.86),
    lat: ui.Textbox('LATITUDE', -33.93),
    locationButton: ui.Button('Select Location', app.selectLocation),
    clearButton: ui.Button('Clear Selection', app.clearSelection)
  };
  
   /* The panel for location selection section with corresponding widgets. */
  app.location.panel = ui.Panel({
    widgets: [
      ui.Label('1) Select location', {fontWeight: 'bold', color: '#9DB68C'}),
      ui.Label('Click to identify a location on the map OR ' +
                  'Manually input a location'),
      ui.Label('Longitude', app.HELPER_TEXT_STYLE), app.location.lon,
      ui.Label('Latitude', app.HELPER_TEXT_STYLE), app.location.lat,
      ui.Panel([
        app.location.locationButton, app.location.clearButton
      ], ui.Panel.Layout.flow('horizontal'))
    ],
    style: app.SECTION_STYLE
  });
  
  /* The image picker section. */
  app.imagery = {
    l8: ui.Checkbox({label: 'Landsat 8', value: false}),
    modis: ui.Checkbox({label: 'MODIS', value: false})
  };

  /* The panel for the image picker section with corresponding widgets. */
  app.imagery.panel = ui.Panel({
    widgets: [
      ui.Label('2) Select a satellite', {fontWeight: 'bold', color: '#9DB68C'}),
      ui.Panel([
        app.imagery.l8,
        app.imagery.modis
      ], ui.Panel.Layout.flow('horizontal'))
    ],
    style: app.SECTION_STYLE
  });
  
  /* The calculate productivity section. */
  app.grazing = {
    grazingButton: ui.Button('Calculate Productivity', app.calcGrazing),
    loadingLabel: ui.Label({
      value: 'Loading...',
      style: {stretch: 'vertical', color: 'gray', shown: false}
    })
  };

  /* The panel for the calculate productivity section with corresponding widgets. */
  app.grazing.panel = ui.Panel({
    widgets: [
      ui.Label('3) Calculate land use productivity', {fontWeight: 'bold', color: '#9DB68C'}),
      ui.Panel([
        app.grazing.grazingButton,
        app.grazing.loadingLabel
      ], ui.Panel.Layout.flow('horizontal'))
    ],
    style: app.SECTION_STYLE
  });
  
  /* The chart results section. */
  app.results = ui.Panel();

  /* The panel for the chart results section with corresponding widgets. */
  app.results.panel = ui.Panel({
    widgets: [
      ui.Label('4) Chart productivity over time', {fontWeight: 'bold', color: '#9DB68C'}),
      ui.Label('Click on the map to generate an NPP time series chart.'),
      app.results
    ],
    style: app.SECTION_STYLE
  });

  /* The export section. */
  app.export = {
    button: ui.Button('Export to Drive', app.export),
  };

  /* The panel for the export section with corresponding widgets. */
  app.export.panel = ui.Panel({
    widgets: [
      ui.Label('5) Export results to Google Drive', {fontWeight: 'bold', color: '#9DB68C'}),
      app.export.button
    ],
    style: app.SECTION_STYLE
  });
  
   
  /* The inspector section. */
  app.inspector = {
    grazeLS: ui.Textbox({placeholder: 'GRAZING CAPACITY: LS 8', value: '0 per/LSU', disabled: true}),
    grasscoverLS: ui.Textbox({placeholder: 'GRASS COVER: LS 8', value: '0 %', disabled: true}),
    grazeMODIS: ui.Textbox({placeholder: 'GRAZING CAPACITY: MODIS', value: '0 per/LSU', disabled: true}),
    grasscoverMODIS: ui.Textbox({placeholder: 'GRASS COVER: MODIS', value: '0 %', disabled: true}),
    subpanel: ui.Panel(),
    loadingLabel: ui.Label({ value: 'Click to inspect productivity values...', style: {stretch: 'vertical', color: 'gray', shown: false }})
    
  };
  
  /* The panel for inspector section displaying Landsat 8 results. */
  app.inspector.panelLS = ui.Panel({
    widgets: [
      ui.Label('Landsat 8', {fontWeight: 'bold', position: 'top-center', color: '#9DB68C'}),
      ui.Panel({
        widgets: [
          ui.Panel({
            widgets: [
              ui.Label('Grass Cover (%):', app.HELPER_TEXT_STYLE), app.inspector.grasscoverLS,
            ],
            layout: ui.Panel.Layout.flow('vertical')
          }),
          ui.Panel({
            widgets: [
              ui.Label('Grazing Capacity (ha/LSU):', app.HELPER_TEXT_STYLE), app.inspector.grazeLS,
            ],
            layout: ui.Panel.Layout.flow('vertical')
          })
        ], 
        layout: ui.Panel.Layout.flow('horizontal')
      }),
    ], 
    layout: ui.Panel.Layout.flow('vertical'),
    style: {textAlign: 'center', stretch: 'vertical'}
  });
  
  /* The panel for inspector section displaying MODIS results. */
  app.inspector.panelMODIS = ui.Panel({
    widgets: [
      ui.Label('MODIS', {fontWeight: 'bold', color: '#9DB68C'}),
      ui.Panel({
        widgets: [
          ui.Panel({
            widgets: [
              ui.Label('Grass Cover (%):', app.HELPER_TEXT_STYLE), app.inspector.grasscoverMODIS,
            ],
            layout: ui.Panel.Layout.flow('vertical')
          }),
          ui.Panel({
            widgets: [
              ui.Label('Grazing Capacity (ha/LSU):', app.HELPER_TEXT_STYLE), app.inspector.grazeMODIS,
            ],
            layout: ui.Panel.Layout.flow('vertical')
          })
        ], 
        layout: ui.Panel.Layout.flow('horizontal')
      }),
    ], 
    layout: ui.Panel.Layout.flow('vertical')
  });
  
  /* The panel for the inspector section with corresponding widgets. */
  app.inspector.panel = ui.Panel({
    widgets: [
        app.inspector.loadingLabel,
        app.inspector.subpanel
      ],
      layout: ui.Panel.Layout.flow('vertical'),
      style: {position: 'top-left'}
  });
  
};

/** Creates the app helper functions. */
app.createHelpers = function() {
  /**
   * Enables or disables loading mode.
   * @param {boolean} enabled Whether loading mode is enabled.
   */
  app.setLoadingMode = function(enabled) {
    // Set each of the widgets to the given enabled mode.
    var loadDependentWidgets = [
      app.location.lon,
      app.location.lat,
      app.location.locationButton,
      app.imagery.l8,
      app.imagery.modis,
      app.grazing.grazingButton,
    ];
    loadDependentWidgets.forEach(function(widget) {
      widget.setDisabled(enabled);
    });
  };
  
  /** Select the current location identified by the cursor. */
  app.selectLocation = function() {
    // Reset all layers and parameters.
    Map.layers().reset();
    app.inspector.subpanel.clear();
    app.results.clear();
    app.imagery.l8.setValue(false);
    app.imagery.modis.setValue(false);
    
    // Get cursor longitude and latitiude.
    finalLon = app.location.lon.getValue();
    finalLat = app.location.lat.getValue();
    calcl8 = false;
    calcmodis = false;
    Map.setCenter(finalLon, finalLat, app.ZOOM);
  };
  
  /** Clear selection and reset the map and UI. */
  app.clearSelection = function() {
    // Reset all layers and parameters.
    Map.layers().reset();
    app.inspector.subpanel.clear();
    app.results.clear();
    app.imagery.l8.setValue(false);
    app.imagery.modis.setValue(false);
    
    // Get cursor longitude and latitiude.
    finalLon = app.location.lon.setValue(18.86);
    finalLat = app.location.lat.setValue(-33.93);
    calcl8 = false;
    calcmodis = false;
    Map.setCenter(finalLon, finalLat, app.ZOOM);
  };
  
  /** Calculate NPP using Landsat 8. */
  app.nppL8 = function() {
    var npp = tools.nppLS(app.COLLECTION_ID_L8, ee.Geometry.Point([finalLon, finalLat]), finalLat ,'2018-02-18', '2019-02-18');
    return npp;
  };
  
  /** Calculate NPP using MODIS. */
  app.nppMODIS = function() {
    var npp = tools.nppMODIS(app.COLLECTION_ID_MODIS, ee.Geometry.Point([finalLon, finalLat]), finalLat ,'2018-02-18', '2019-02-18');
    return npp;
  };
  
  /** Calculate grazing capacity. */
  app.gc = function(npp) {
    var aNPP = npp.sum();
  
    var avg_kg = 400;
    
    var an_consump = ((2.5 / 100) * 400) * 365;
    var LSU = aNPP.expression(
          '( cons / (0.3 * aGPP))', {
            'aGPP': aNPP.select("NPP"),
            'cons': an_consump
        }).rename('LSU');
    return(LSU);
  };
  
  /** Calculate grazing capacity using Landsat 8. */
  app.grazingL8 = function() {
      var graze = tools.grazingCapLS(app.COLLECTION_ID_L8, ee.Geometry.Point([finalLon, finalLat]), finalLat ,'2018-02-18', '2019-02-18');
      print(graze)
      return graze;
  };
  
  /** Calculate grass cover using Landsat 8. */
  app.grassL8 = function() {
      var grassCov = tools.grassCoverLS(app.COLLECTION_ID_L8, ee.Geometry.Point([finalLon, finalLat]), startDate, endDate);
      print(grassCov)
      return grassCov;
  };
  
  /** Calculate grazing capacity using MODIS. */
  app.grazingMODIS = function() {
      var graze = tools.grazingCapMODIS(app.COLLECTION_ID_MODIS, ee.Geometry.Point([finalLon, finalLat]), finalLat ,'2018-02-18', '2019-02-18');
      print(graze)
      return graze;
  };
  
  /** Calculate grass cover using MODIS. */
  app.grassMODIS = function() {
    var grassCov = tools.grassCoverMODIS(app.COLLECTION_ID_MODIS, ee.Geometry.Point([finalLon, finalLat]), startDate, endDate);
    print(grassCov)
    return grassCov;
  };
  
  /** Method to run when "Calculate Productivity" button clicked. */
  app.calcGrazing = function() {
    
    print("Grazing calc");
    calcl8 = app.imagery.l8.getValue();
    calcmodis = app.imagery.modis.getValue();
    
    
    if (calcl8 && calcmodis ) {
      var grassCovL8 = app.grassL8();
      var grassCovMODIS = app.grassMODIS();
      var nppL8 = app.nppL8();
      var grazeL8 = app.gc(nppL8);
      var nppMODIS = app.nppMODIS();
      var grazeMODIS = app.gc(nppMODIS);
      Map.addLayer(ee.Image(grassCovL8), app.VIS_GRASSCOVER, 'Grass Cover (%): Landsat 8');
      Map.addLayer(ee.Image(grassCovMODIS), app.VIS_GRASSCOVER, 'Grass Cover (%): MODIS');
      Map.addLayer(ee.Image(grazeL8), app.VIS_GRAZINGCAP, 'Grazing Capacity (ha/LSU): Landsat 8');
      Map.addLayer(ee.Image(grazeMODIS), app.VIS_GRAZINGCAP, 'Grazing Capacity (ha/LSU): MODIS');
      app.inspector.subpanel.add(app.inspector.panelLS);
      app.inspector.subpanel.add(app.inspector.panelMODIS);
      
      globalNPPL8 = nppL8;
      globalNPPMODIS = nppMODIS;
      
      var chartL8 = app.chartResults(nppL8);
      chartL8.setOptions(app.CHART_OPTIONS_L8);
      app.results.add(chartL8);
      
      var chartMODIS = app.chartResults(nppMODIS);
      chartMODIS.setOptions(app.CHART_OPTIONS_MODIS);
      app.results.add(chartMODIS);
      
    } else if (calcl8) {
      var grassCovL8 = app.grassL8();
      var nppL8 = app.nppL8();
      print(nppL8)
      var grazeL8 = app.gc(nppL8);
      print(grazeL8)
      Map.addLayer(ee.Image(grassCovL8), app.VIS_GRASSCOVER, 'Grass Cover (%): Landsat 8');
      Map.addLayer(ee.Image(grazeL8), app.VIS_GRAZINGCAP, 'Grazing Capacity (ha/LSU): Landsat 8');
      app.inspector.subpanel.add(app.inspector.panelLS);
      
      globalNPPL8 = nppL8;
      
      var chartL8 = app.chartResults(nppL8);
      chartL8.setOptions(app.CHART_OPTIONS_L8);
      app.results.add(chartL8);
      
    } else if (calcmodis) {
      var grassCovMODIS = app.grassMODIS();
      var nppMODIS = app.nppMODIS();
      var grazeMODIS = app.gc(nppMODIS);
      Map.addLayer(ee.Image(grassCovMODIS), app.VIS_GRASSCOVER, 'Grass Cover (%): MODIS');
      Map.addLayer(ee.Image(grazeMODIS), app.VIS_GRAZINGCAP, 'Grazing Capacity (ha/LSU): MODIS');
      
      app.inspector.subpanel.add(app.inspector.panelMODIS);
      
      globalNPPMODIS = nppMODIS;
      
      var chartMODIS = app.chartResults(nppMODIS);
      chartMODIS.setOptions(app.CHART_OPTIONS_MODIS);
      app.results.add(chartMODIS);
    }
    grazeResult = true;
    grassResult = true;
    app.inspector.loadingLabel.style().set('shown', true);
    app.extract();

  };
  
  /** Extract grass cover and grazing capacity values from map layers. */
  app.extract = function() {
    app.setLoadingMode(true);
    var point = ee.Geometry.Point([finalLon, finalLat]);
    
    // Loop through each layer on the map and create an image.
    var allLayers = [];
    var layers = Map.layers();
    layers.forEach(function(lay) {
      var img = ee.Image(lay.getEeObject());
      var name = ee.String(lay.getName());
      print(name, img);
      allLayers.push(img.select(0));
      
    });
    var allLayersImg = ee.Image(allLayers);
    
    // Extract all grass cover and grazing capacity values
    var allExtract = allLayersImg.reduceRegion(ee.Reducer.mean(), point, 30);
    
    // Set inspector panel values accordingly
    if (calcl8 && calcmodis) {
      // Set loading inspector to loading
      app.inspector.grasscoverLS.style().set('color', 'gray');
      app.inspector.grazeLS.style().set('color', 'gray');
      app.inspector.grasscoverMODIS.style().set('color', 'gray');
      app.inspector.grazeMODIS.style().set('color', 'gray');
      app.inspector.grasscoverLS.setValue('Loading...');
      app.inspector.grazeLS.setValue('Loading...');
      app.inspector.grasscoverMODIS.setValue('Loading...');
      app.inspector.grazeMODIS.setValue('Loading...');
      
      // Extract values asynchrononously and display
      allExtract.get("Grass Cover").evaluate(function(result) { app.inspector.grasscoverLS.setValue(result.toFixed(2)); app.inspector.grasscoverLS.style().set('color', 'black');});
      allExtract.get("LSU").evaluate(function(result) { app.inspector.grazeLS.setValue(result.toFixed(2)); app.inspector.grazeLS.style().set('color', 'black');});
      allExtract.get("Grass Cover_1").evaluate(function(result) { app.inspector.grasscoverMODIS.setValue(result.toFixed(2)); app.inspector.grasscoverMODIS.style().set('color', 'black');});
      allExtract.get("LSU_1").evaluate(function(result) { app.inspector.grazeMODIS.setValue(result.toFixed(2)); app.inspector.grazeMODIS.style().set('color', 'black');});
    } else if (calcl8) {
      // Set loading inspector to loading
      app.inspector.grasscoverLS.style().set('color', 'gray');
      app.inspector.grazeLS.style().set('color', 'gray');
      app.inspector.grasscoverLS.setValue('Loading...');
      app.inspector.grazeLS.setValue('Loading...');
      
      // Extract values asynchrononously and display
      allExtract.get("Grass Cover").evaluate(function(result) { app.inspector.grasscoverLS.setValue(result.toFixed(2)); app.inspector.grasscoverLS.style().set('color', 'black');});
      allExtract.get("LSU").evaluate(function(result) { app.inspector.grazeLS.setValue(result.toFixed(2)); app.inspector.grazeLS.style().set('color', 'black');});
    } else if (calcmodis) {
      // Set loading inspector to loading
      app.inspector.grasscoverMODIS.style().set('color', 'gray');
      app.inspector.grazeMODIS.style().set('color', 'gray');
      app.inspector.grasscoverMODIS.setValue('Loading...');
      app.inspector.grazeMODIS.setValue('Loading...');
      
      // Extract values asynchrononously and display
      allExtract.get("Grass Cover").evaluate(function(result) { app.inspector.grasscoverMODIS.setValue(result.toFixed(2)); app.inspector.grasscoverMODIS.style().set('color', 'black');});
      allExtract.get("LSU").evaluate(function(result) { app.inspector.grazeMODIS.setValue(result.toFixed(2)); app.inspector.grazeMODIS.style().set('color', 'black');});
    }
    app.setLoadingMode(false);
    
    
      /*
      //var len = Map.layers().length();
      for (var i = 0; i < len; i++) {
        var img = ee.Image(Map.layers().get(i).getEeObject());
        var name = ee.String(Map.layers().get(i).getName());
        print("Layer:", name);
        print(img);
        print("Grass i", ee.Algorithms.IsEqual(name, ee.String('Grass Cover (%): Landsat 8')));
        print("Graze Eq",  ee.Algorithms.IsEqual(name, ee.String('Grazing Capacity (ha/LSU): Landsat 8')));
        //if ((name.index('Grass Cover (%): Landsat 8')).getInfo() > -1)
        if (calcl8 && calcmodis ) {
          if ((name.index('Grass Cover (%): Landsat 8')).getInfo() > -1) {
            var sampledPoint = img.reduceRegion(ee.Reducer.mean(), point, 30);
            var computedValue = sampledPoint.get("Grass Cover");
            //app.inspector.grasscoverLS.setValue(computedValue.getInfo());
            computedValue.evaluate(function(result) { app.inspector.grasscoverLS.setValue(result.toFixed(2));});
          } else if ((name.index('Grazing Capacity (ha/LSU): Landsat 8')).getInfo() > -1) {
            var sampledPoint = img.reduceRegion(ee.Reducer.mean(), point, 30);
            var computedValue = sampledPoint.get("LSU");
            //app.inspector.grazeLS.setValue(computedValue.getInfo());
            computedValue.evaluate(function(result) { app.inspector.grazeLS.setValue(result.toFixed(2));});
          } else if ((name.index('Grass Cover (%): MODIS')).getInfo() > -1) {
            var sampledPoint = img.reduceRegion(ee.Reducer.mean(), point, 30);
            var computedValue = sampledPoint.get("Grass Cover");
            //app.inspector.grasscoverMODIS.setValue(computedValue.getInfo());
            computedValue.evaluate(function(result) { app.inspector.grasscoverMODIS.setValue(result.toFixed(2));});
          } else if ((name.index('Grazing Capacity (ha/LSU): MODIS')).getInfo() > -1) {
            var sampledPoint = img.reduceRegion(ee.Reducer.mean(), point, 30);
            var computedValue = sampledPoint.get("LSU");
            //app.inspector.grazeMODIS.setValue(computedValue.getInfo());
            computedValue.evaluate(function(result) { app.inspector.grazeMODIS.setValue(result.toFixed(2));});
          }
          
        } else if (calcl8) {
          if (ee.Algorithms.IsEqual(name, ee.String('Grass Cover (%): Landsat 8'))) {
            var sampledPoint = img.reduceRegion(ee.Reducer.mean(), point, 30);
            var computedValue = sampledPoint.get("Grass Cover");
            print('grass comp', computedValue);
            //app.inspector.grasscoverLS.setValue(computedValue.getInfo());
            computedValue.evaluate(function(result) { app.inspector.grasscoverLS.setValue(result.toFixed(2));});
          } else if (ee.Algorithms.IsEqual(name, ee.String('Grazing Capacity (ha/LSU): Landsat 8'))) {
            var sampledPoint = img.reduceRegion(ee.Reducer.mean(), point, 30);
            var computedValue = sampledPoint.get("LSU");
            //app.inspector.grazeLS.setValue(computedValue.getInfo());
            print('graze comp', computedValue);
            computedValue.evaluate(function(result) { app.inspector.grazeLS.setValue(result.toFixed(2));});
          }
          
        } else if (calcmodis) { 
          if ((name.index('Grass Cover')).getInfo() > -1) {
            var sampledPoint = img.reduceRegion(ee.Reducer.mean(), point, 30);
            var computedValue = sampledPoint.get("Grass Cover");
            //app.inspector.grasscoverMODIS.setValue(computedValue.getInfo());
            computedValue.evaluate(function(result) { app.inspector.grasscoverMODIS.setValue(result.toFixed(2));});
          } else if ((name.index('Grazing Capacity')).getInfo() > -1) {
            var sampledPoint = img.reduceRegion(ee.Reducer.mean(), point, 30);
            var computedValue = sampledPoint.get("LSU");
            //app.inspector.grazeMODIS.setValue(computedValue.getInfo());
            computedValue.evaluate(function(result) { app.inspector.grazeMODIS.setValue(result.toFixed(2));});
          }
        }
      }*/
      //app.inspector.loadingLabel.setValue('Click to inspect productivity values...');
  };
  
  /** Chart annual NPP. */
  app.chartResults = function(timeSeries) {
    var point = ee.Geometry.Point([finalLon, finalLat]);
    var chart = ui.Chart.image.series({
      imageCollection: timeSeries.select('NPP'), 
      region: point, 
      reducer: ee.Reducer.mean(), 
      scale: 30
    });
    return chart;
  };
  
  /** Export grass cover and grazing capacity results to Google Drive. */
  app.export = function() {
    // Loop through each layer on the map, create an image and export
    var layers = Map.layers();
    layers.forEach(function(lay) {
      var img = ee.Image(lay.getEeObject());
      var name = ee.String(lay.getName()).getInfo();
      var region = img.geometry(10).bounds(10).getInfo()["coordinates"];
      var imgtype = img.toFloat();
      
      Export.image.toDrive({
        image: imgtype,
        description: name,
        fileNamePrefix: name,
        region: region,
        scale: 30,
        maxPixels: 1e10})
    });
  };
  
};

/** Creates the app constants. */
app.createConstants = function() {
  app.LON = 18.864008813421833;
  app.LAT = -33.93500608751589;
  app.ZOOM = 13;
  app.COLLECTION_ID_L8 = 'LANDSAT/LC08/C01/T1_TOA';
  app.COLLECTION_ID_MODIS = 'MODIS/006/MOD09A1';
  app.SECTION_STYLE = {margin: '20px 0 0 0'};
  app.HELPER_TEXT_STYLE = {
      margin: '8px 0 -3px 8px',
      fontSize: '12px',
      color: 'gray'
  };
  app.IMAGE_COUNT_LIMIT = 10;
  app.VIS_GRASSCOVER = { min: 0, max: 100, palette: ['#6A462C', '#AC886E', '#ECCBB3', '#D5ECB3', '#ABC484', '#839862']};
  app.VIS_GRAZINGCAP = { min: 0, max: 20, palette: ['green', 'yellow', 'orange', 'red']}; 
  app.CHART_OPTIONS_L8 = {
    title: 'Annual NPP for Landsat 8',
    vAxis: {title: 'NPP'},
    hAxis: {title: 'date', format: 'MM-yy', gridlines: {count: 7}},
    interpolateNulls: true,
    series: {
      0: {color: '#78AB46'}
    }
  }
  app.CHART_OPTIONS_MODIS = {
    title: 'Annual NPP for MODIS',
    vAxis: {title: 'NPP'},
    hAxis: {title: 'date', format: 'MM-yy', gridlines: {count: 7}},
    interpolateNulls: true,
    series: {
      0: {color: '#78AB46'}
    }
  }
  
  app.CUSTOM_BASE_MAP_STYLE = { 
    'Light Theme': [
    {
        "featureType": "all",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "administrative",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "administrative.country",
        "elementType": "geometry.stroke",
        "stylers": [
            {
                "visibility": "on"
            }
        ]
    },
    {
        "featureType": "administrative.country",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "on"
            }
        ]
    },
    {
        "featureType": "administrative.province",
        "elementType": "geometry.stroke",
        "stylers": [
            {
                "visibility": "on"
            },
            {
                "color": "#000000"
            }
        ]
    },
    {
        "featureType": "administrative.province",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "on"
            }
        ]
    },
    {
        "featureType": "administrative.locality",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "on"
            }
        ]
    },
    {
        "featureType": "landscape",
        "elementType": "geometry",
        "stylers": [
            {
                "visibility": "on"
            },
            {
                "color": "#e3e3e3"
            }
        ]
    },
    {
        "featureType": "landscape",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "landscape.man_made",
        "elementType": "geometry.fill",
        "stylers": [
            {
                "saturation": "0"
            },
            {
                "lightness": "6"
            },
            {
                "weight": "0.90"
            }
        ]
    },
    {
        "featureType": "landscape.man_made",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "landscape.natural",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "poi",
        "elementType": "all",
        "stylers": [
            {
                "visibility": "off"
            },
            {
                "lightness": "0"
            }
        ]
    },
    {
        "featureType": "poi",
        "elementType": "geometry.fill",
        "stylers": [
            {
                "visibility": "simplified"
            }
        ]
    },
    {
        "featureType": "poi",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "poi.attraction",
        "elementType": "all",
        "stylers": [
            {
                "visibility": "simplified"
            }
        ]
    },
    {
        "featureType": "poi.attraction",
        "elementType": "geometry",
        "stylers": [
            {
                "visibility": "simplified"
            }
        ]
    },
    {
        "featureType": "poi.attraction",
        "elementType": "geometry.fill",
        "stylers": [
            {
                "visibility": "simplified"
            }
        ]
    },
    {
        "featureType": "poi.attraction",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "poi.business",
        "elementType": "all",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "poi.business",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "poi.government",
        "elementType": "all",
        "stylers": [
            {
                "visibility": "simplified"
            }
        ]
    },
    {
        "featureType": "poi.government",
        "elementType": "geometry",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "poi.government",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "poi.medical",
        "elementType": "all",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "poi.park",
        "elementType": "geometry.fill",
        "stylers": [
            {
                "visibility": "on"
            },
            {
                "color": "#c7d7af"
            },
            {
                "lightness": "-14"
            }
        ]
    },
    {
        "featureType": "poi.place_of_worship",
        "elementType": "all",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "poi.school",
        "elementType": "all",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "poi.sports_complex",
        "elementType": "all",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "road",
        "elementType": "all",
        "stylers": [
            {
                "color": "#cccccc"
            }
        ]
    },
    {
        "featureType": "road",
        "elementType": "geometry",
        "stylers": [
            {
                "visibility": "on"
            }
        ]
    },
    {
        "featureType": "road",
        "elementType": "geometry.fill",
        "stylers": [
            {
                "saturation": "0"
            },
            {
                "lightness": "30"
            }
        ]
    },
    {
        "featureType": "road",
        "elementType": "geometry.stroke",
        "stylers": [
            {
                "visibility": "on"
            }
        ]
    },
    {
        "featureType": "road",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "transit",
        "elementType": "all",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "transit",
        "elementType": "labels.icon",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "transit.line",
        "elementType": "geometry",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "transit.line",
        "elementType": "labels.text",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "transit.station",
        "elementType": "all",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "transit.station.airport",
        "elementType": "geometry",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "transit.station.airport",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "water",
        "elementType": "geometry",
        "stylers": [
            {
                "color": "#FFFFFF"
            }
        ]
    },
    {
        "featureType": "water",
        "elementType": "geometry.fill",
        "stylers": [
            {
                "lightness": "-28"
            },
            {
                "saturation": "33"
            },
            {
                "color": "#c7def0"
            }
        ]
    },
    {
        "featureType": "water",
        "elementType": "labels",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    }
]};
  
  app.VIS_OPTIONS = {
    'False color (B12/B11/B4)': {
      description: 'Vegetation is shades of red, urban areas are ' +
                   'cyan blue, and soils are browns.',
      visParams: {gamma: 1, max: 8048, bands: ['B12', 'B11', 'B4']}
    },
    'Natural color (B4/B3/B2)': {
      description: 'Ground features appear in colors similar to their ' +
                   'appearance to the human visual system.',
      visParams: {gamma: 1, max: 8048, bands: ['B4', 'B3', 'B2']}
    },
    'Atmospheric (B12/B11/B8)': {
      description: 'Coast lines and shores are well-defined. ' +
                   'Vegetation appears blue.',
      visParams: {gamma: 1, max: 8048, bands: ['B12', 'B11', 'B8']}
    }
  };
};

/** Creates the application interface. */
app.boot = function() {
  app.createConstants();
  app.createHelpers();
  app.createPanels();
  var main = ui.Panel({
    widgets: [
      app.intro.panel,
      app.location.panel,
      app.imagery.panel,
      app.grazing.panel,
      app.results.panel,
      app.export.panel,
    ],
    style: {width: '320px', padding: '8px'}
  });
  Map.add(app.inspector.panel);
  app.inspector.panel.style().set('shown', true);
  
  Map.setCenter(app.LON, app.LAT, app.ZOOM);
  Map.setOptions('Light Theme' , app.CUSTOM_BASE_MAP_STYLE);
  Map.style().set('cursor', 'crosshair');
  
  // Update isnpector and location when map clicked
  Map.onClick(function(coords) {
    app.location.lon.setValue(coords.lon);
    app.location.lat.setValue(coords.lat);
    finalLon = coords.lon;
    finalLat = coords.lat;
    
   if (grazeResult && grassResult) {
      app.extract();
      
      app.results.clear();
      if (calcl8 && calcmodis) {
        var chartL8 = app.chartResults(globalNPPL8);
        chartL8.setOptions(app.CHART_OPTIONS_L8);
        app.results.add(chartL8);
        
        var chartMODIS = app.chartResults(globalNPPMODIS);
        chartMODIS.setOptions(app.CHART_OPTIONS_MODIS);
        app.results.add(chartMODIS);
        
      } else if (calcl8) {
        var chartL8 = app.chartResults(globalNPPL8);
        chartL8.setOptions(app.CHART_OPTIONS_L8);
        app.results.add(chartL8);
        
      } else if (calcmodis) {
        var chartMODIS = app.chartResults(globalNPPMODIS);
        chartMODIS.setOptions(app.CHART_OPTIONS_MODIS);
        app.results.add(chartMODIS);
      }
    }
  });
  ui.root.insert(0, main);
};

app.boot();
