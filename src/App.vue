<template>
 <v-app>
  <v-app-bar app
    absolute>
    <v-app-bar-title>室内地图制作</v-app-bar-title>
    <v-tabs align-with-title v-model="drawType">
      <v-tab>区域</v-tab>
      <v-tab>路线</v-tab>
      <v-tab>POI</v-tab>
    </v-tabs>
    <v-btn @click="saveGeoJson">下载</v-btn>
 
    <!-- <v-btn @click="isLayerDrawer = true">图层</v-btn> -->
    <input type="file" accept="text/geojson" style="display:none" ref="file_input" @change="fileChoosed" />
    <v-btn @click="chooseFile" >导入</v-btn>




  </v-app-bar>
  <v-main app>
    <v-layout>
    <v-row class="map" justify="center">
      <v-col
        class="mb-1"
        cols="12">
        <l-map ref="map" :zoom="zoom" :center="center">
          <l-tile-layer :url="url" attribution="地图制作" />
        </l-map>
      </v-col>
    </v-row>
  </v-layout>
  </v-main>
  <v-navigation-drawer
    width=356
    v-model="isLayerDrawer"
    right
    absolute
    temporary >
    <v-list dense>
      <v-subheader>图层列表</v-subheader>
       <v-list-item-group
        color="primary"
      >
        <v-list-item
          v-for="(item, i) in layers"
          :key="i" @click="onLayerClick(item)"
        >
          <v-list-item-content>
            <v-list-item-title v-text="item.options.title? item.options.title : item.options.id"></v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list-item-group>
    </v-list>
  </v-navigation-drawer>
  <v-navigation-drawer
  v-if="selectedLayer && selectedLayer.internalId"
      width=356
      absolute
      permanent
      right app>
   <v-banner
    color="primary">
    详细信息
    <v-btn style="float:right"  @click="saveAreaStyle">保存</v-btn>
    </v-banner>
     <v-container v-if="selectedLayer && selectedLayer.internalId" style="margin-left:12px;">
       <v-row>
         <v-text-field
            v-model="selectedLayer.options.title"
            label="名称"
            required
          ></v-text-field>
       </v-row>
       <template v-if="selectedLayer.shape == 'Marker'" >
        <v-row>
         <v-select
          v-model="selectIcon"
          :items="icons"
          label="选择图标"
        ></v-select>
       </v-row>
       </template>
      <template v-if="selectedLayer.shape == 'Line'" >
          <v-row>
          <v-card>
            <v-card-text>
              路线颜色
              <v-color-picker v-model="selectedLayer.options.color" flat />
            </v-card-text>
          </v-card>
        </v-row>
        <v-row>
          <v-text-field
            v-model="selectedLayer.options.weight"
            label="路线宽度"
            required
          ></v-text-field>
        </v-row>
       </template>
       <template v-if="selectedLayer.shape == 'Polygon' || selectedLayer.shape == 'Rectangle'">
        <v-row>
          <v-card>
            <v-card-text>
              请选择填充颜色
              <v-color-picker v-model="selectedLayer.options.fillColor" flat />
            </v-card-text>
          </v-card>
        </v-row>
        <v-row>
          <v-text-field
            v-model="selectedLayer.options.fillOpacity"
            label="填充透明度"
            required
          ></v-text-field>
        </v-row>
        <v-row>
          <v-card>
            <v-card-text>
              请选择边框颜色
              <v-color-picker v-model="selectedLayer.options.color" flat />
            </v-card-text>
          </v-card>
        </v-row>
        <v-row>
          <v-text-field
            v-model="selectedLayer.options.weight"
            label="边框宽度"
            required
          ></v-text-field>
        </v-row>
        
       </template>
       
     </v-container>
  </v-navigation-drawer>

  <v-footer >
    
  </v-footer>
</v-app>
</template>

<script>
import L, { Icon } from 'leaflet'
import '@geoman-io/leaflet-geoman-free';
import { v4 as uuidv4 } from 'uuid';
import {LMap,LTileLayer} from 'vue2-leaflet'
export default {
  name: 'App',

  components: {
    LMap,
    LTileLayer
  },

  data: () => ({
     items: ['Foo', 'Bar', 'Fizz', 'Buzz'],
    isLayerDrawer: false,
    url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
    zoom: 12,
    center: {lat: 39.1136057, lng: 116.9780756},
    leafletGeoman: null,
    layers:[],
    drawType: 0,
    selectedLayer: {
      
    },
    selectIcon: null,
    icons: ["airfield-11","airfield-15","airport-11","airport-15","alcohol-shop-11","alcohol-shop-15","america-football-11","america-football-15","amusement-park-11","amusement-park-15","aquarium-11","aquarium-15","art-gallery-11","art-gallery-15","attraction-11","attraction-15","bakery-11","bakery-15","bank-11","bank-15","bar-11","bar-15","baseball-11","baseball-15","basketball-11","basketball-15","beer-11","beer-15","bicycle-11","bicycle-15","bicycle-share-11","bicycle-share-15","bus-11","bus-15","cafe-11","cafe-15","campsite-11","campsite-15","car-11","car-15","castle-11","castle-15","cemetery-11","cemetery-15","cinema-11","cinema-15","circle-11","circle-15","circle-stroked-11","circle-stroked-15","clothing-store-11","clothing-store-15","college-11","college-15","dentist-11","dentist-15","doctor-11","doctor-15","dog-park-11","dog-park-15","drinking-water-11","drinking-water-15","embassy-11","embassy-15","fast-food-11","fast-food-15","ferry-11","ferry-15","fire-station-11","fire-station-15","fuel-11","fuel-15","garden-11","garden-15","golf-11","golf-15","grocery-11","grocery-15","hairdresser-11","hairdresser-15","harbor-11","harbor-15","heliport-11","heliport-15","hospital-11","hospital-15","ice-cream-11","ice-cream-15","information-11","information-15","laundry-11","laundry-15","library-11","library-15","lodging-11","lodging-15","marker-11","marker-15","monument-11","monument-15","mountain-11","mountain-15","museum-11","museum-15","music-11","music-15","park-11","park-15","pharmacy-11","pharmacy-15","picnic-site-11","picnic-site-15","place-of-worship-11","place-of-worship-15","playground-11","playground-15","police-11","police-15","post-11","post-15","prison-11","prison-15","rail-11","rail-15","rail-light-11","rail-light-15","rail-metro-11","rail-metro-15","religious-christian-11","religious-christian-15","religious-jewish-11","religious-jewish-15","religious-muslim-11","religious-muslim-15","restaurant-11","restaurant-15","rocket-11","rocket-15","school-11","school-15","shop-11","shop-15","stadium-11","stadium-15","star-11","star-15","suitcase-11","suitcase-15","sushi-11","sushi-15","swimming-11","swimming-15","theatre-11","theatre-15","toilet-11","toilet-15","town-hall-11","town-hall-15","triangle-11","triangle-15","triangle-stroked-11","triangle-stroked-15","veterinary-11","veterinary-15","volcano-11","volcano-15","zoo-11","zoo-15"],
    mapDrawOptions: {
      snappable: false,
      snapDistance: 20,
      allowSelfIntersection: false,
      finishOn: 'dblclick',
      // templineStyle: {
      //     color: 'black',
      //     dashArray: [10,10],
      //     weight: 3
      // },
      // hintlineStyle: {
      //     color: '#7A797A',
      //     dashArray: [10,10],
      // },
      pathOptions: {
          color: '#1976D2FF',
          fillColor: '#1976D2FF',
          weight: 2,
          fillOpacity: 0.2,
          opacity: 1
      }
    },
  }),
  created() {
      
  },
  watch: {
    drawType(){
      this.creatControls();
      if(this.drawType == 0){
        this.leafletGeoman.pm.enableDraw('Polygon');
      }else if(this.drawType == 1) {
        this.leafletGeoman.pm.enableDraw('Line');
      }else{
        this.leafletGeoman.pm.enableDraw('Marker');
      }
    }
  },
  mounted () {
    this.leafletGeoman = this.$refs.map.mapObject;
    this.leafletGeoman.pm.setGlobalOptions({ allowSelfIntersection: false });
    this.leafletGeoman.pm.setLang(this.center);
    this.leafletGeoman.pm.setLang('zh');
    this.leafletGeoman.pm.enableDraw('Polygon', this.mapDrawOptions);
    this.leafletGeoman.pm.enableDraw('Rectangle', this.mapDrawOptions);
    this.leafletGeoman.pm.disableGlobalEditMode();
    //console.log(this.leafletGeoman);
    this.leafletGeoman._panes.shadowPane.style.display = "none";
    this.creatControls();
    var _that = this;
    this.leafletGeoman.on('pm:create', (e) => {
      if (e.shape == "Marker") {
        _that.leafletGeoman.pm.disableGlobalEditMode();
      }
      e.layer.internalId = uuidv4()
      e.layer.options.id = e.layer.internalId;
      e.layer.options.title = "";
      e.layer.options.description = "";
      e.layer.shape = e.shape
      e.layer.options.shape = e.shape
      _that.layers.push(e.layer);
      //console.log("----",e.layer);
      e.layer.on('click', (l) => {
        //console.log("----",l);
        _that.selectedLayer = l.sourceTarget;
      });
      _that.selectedLayer = e.layer;
    });

  },
  methods: {
    creatControls() {
      this.leafletGeoman.pm.addControls(
        {
        // position: 'topright',
        drawMarker: this.drawType == 2 ? true : false,
        drawCircleMarker: false,
        drawPolyline: this.drawType == 1 ? true : false,
        drawRectangle: this.drawType == 0 ? true : false,
        drawPolygon: this.drawType == 0 ? true : false,
        drawCircle: this.drawType == 0 ? true : false,
        editMode: true,
        dragMode: true,
        cutPolygon: true,
        removalMode: true,
        }
      );
    },
    getAllDataToGeoJSON () {
      const geoJSON = {
        type: 'FeatureCollection',
        features: []
      };
      this.leafletGeoman.eachLayer(function (layer) {
        if (layer.internalId && (layer instanceof L.Path || layer instanceof L.Marker)) {
          const geoJSONShape = layer.toGeoJSON(16); 
          geoJSONShape.properties = layer.options;
          geoJSONShape.id = layer.internalId;
          geoJSON.features.push(geoJSONShape);
        }
      });
      return geoJSON;
    },
    // inport data from GeoJSON
    importGeoJSON (geoJSON) {
      if (geoJSON && geoJSON.type === 'FeatureCollection' && geoJSON.features && geoJSON.features.length) {
        geoJSON.features.forEach(feature => {
          const shape = feature.geometry && feature.geometry.type;
          const coordinates = feature.geometry && feature.geometry.coordinates;
          let layer; // define shape for later use
          console.log(feature);
          switch (shape) {
            case 'Circle':
              layer = new L.Circle([coordinates[1], coordinates[0]], feature.properties.radius);
              break;
            case 'CircleMarker':
              layer = new L.CircleMarker([coordinates[1], coordinates[0]]);
              break;
            case 'Point':
              layer = new L.Marker([coordinates[1], coordinates[0]],{icon: new Icon({iconUrl:feature.properties.icon})});
              break;
            case 'LineString':
              layer = new L.Polyline(this.switchCoordinates([coordinates]));
              layer.setStyle(feature.properties);
              break;
            case 'Polygon':
              layer = new L.Polygon(this.switchCoordinates(coordinates));
              layer.setStyle(feature.properties);
              break;
            case 'Rectangle':
              layer = new L.Rectangle(this.switchCoordinates(coordinates));
              layer.setStyle(feature.properties);
              break;
          }

          if (layer) {
            // console.log(layer);
            layer.on('click', (l) =>{
              this.selectedLayer = l.sourceTarget;
            });

            layer.internalId = feature.id;
            layer.properties = feature.properties;

            if (layer?.properties?.color) {
              layer.setStyle({color: layer.properties.color})
            }
            this.leafletGeoman.addLayer(layer);
            
          }
        });
      }
    },
    switchCoordinates (coordinates) {
      return [coordinates[0].map(pair => [pair[1], pair[0]])];
    },
    saveGeoJson() {
      var Link = document.createElement('a');
      Link.download = "geojson.geojson";
      Link.style.display = 'none';
      const json = this.getAllDataToGeoJSON();
      const jsonStr = JSON.stringify(json);
      var blob = new Blob([jsonStr]);
      Link.href = URL.createObjectURL(blob);
      document.body.appendChild(Link);
      Link.click();
      document.body.removeChild(Link);

    },
    saveAreaStyle() {
      this.selectedLayer.bindTooltip(
            `<strong>${this.selectedLayer.options.title}</strong>`
      )
      if (this.selectedLayer.shape == "Marker" && this.selectIcon) {
        const url = "./_svg/" + this.selectIcon + ".svg";
        //console.log(url);
        this.selectedLayer._icon.src = url;
        this.selectedLayer.options.icon = url;
        //console.log(this.selectedLayer);
        return
      }
      this.selectedLayer.setStyle(
      {
        fillColor: this.selectedLayer.options.fillColor,
        color: this.selectedLayer.options.color,
        weight: this.selectedLayer.options.weight,
        fillOpacity: this.selectedLayer.options.fillOpacity,
      })
      this.selectedLayer = null;
    },
    onLayerClick(item){
      this.selectedLayer = item;
    },
    chooseFile() {
      this.$refs.file_input.dispatchEvent(new MouseEvent('click'))
    },
    fileChoosed() {
      const file = this.$refs.file_input.files[0];
      console.log(file);
      if (!file.name.endsWith('.geojson')) {
        return;
      }
      const reader = new FileReader();
      reader.onload = () => {
        var json = JSON.parse(reader.result);
        this.importGeoJSON(json);
      };
      reader.readAsText(file);

    }
  }
  
  
};
</script>
<style>
 .map {
   height: 95vh;
   width: 100wv;
 }
</style>
<style lang="sass">
    @import '~@geoman-io/leaflet-geoman-free/dist/leaflet-geoman.css'
</style>