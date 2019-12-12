<template>
  <v-app id="inspire">
    <v-navigation-drawer v-model="siteVis" app clipped right>
      <v-list dense>
        <!-- <v-list-item @click.stop="right = !right"> -->

        <!-- <v-list-item-content>
            <v-list-item-title>Open Temporary Drawer</v-list-item-title>
        </v-list-item-content>-->
        <!-- </v-list-item> -->
      </v-list>
      <v-card>
                <v-card-title
                  class="subheading font-weight-bold"
                >CDR Site ID : {{this.curSelSite.propID}}</v-card-title>

                <v-divider></v-divider>
                <v-list dense>
                  <v-list-item v-for="(v,k,index) in this.curSelSite" v-bind:key="index">
                    <v-list-item-content class="caption">{{k}}:</v-list-item-content>
                    <v-list-item-content class="align-end font-weight-thin">{{v}}</v-list-item-content>
                  </v-list-item>
                </v-list>
              </v-card> 
    </v-navigation-drawer>

    <v-app-bar app clipped-right color="grey" dark>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>RATS Infographic MAPS</v-toolbar-title>
      <div class="flex-grow-1"></div>
      <!-- <v-app-bar-nav-icon @click.stop="drawerRight = !drawerRight"></v-app-bar-nav-icon> -->
      
    </v-app-bar>

    <v-navigation-drawer v-model="drawer" app>
      <v-list dense>
        <v-list-item @click="methodThatForcesUpdate">
          <v-list-item-action>
            <v-icon>mdi-home</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Home</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-divider></v-divider>
        <v-expansion-panels popout>
          <v-expansion-panel>
            <v-expansion-panel-header>Revenue Layer</v-expansion-panel-header>
            <v-expansion-panel-content>
              <revenue @toggleRevenueLayer="toggleRevenueLayer"></revenue>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-list>
    </v-navigation-drawer>

    <v-content>
      <v-container class="fill-height" fluid>
        <v-row justify="center" align="center">
          <v-col class="flex-grow-1">
            <v-map :zoom.sync="zoom" :center="initialLocation" ref="map">
            <v-control-layers />
            <v-tilelayer
              v-for="tileProvider in tileProviders"
              :key="tileProvider.name"
              :name="tileProvider.name"
              :visible="tileProvider.visible"
              :url="tileProvider.url"
              :attribution="tileProvider.attribution"
              layer-type="base"
            ></v-tilelayer>
            <v-canvas-layer
              :locations="locations"
              v-if="this.revLayerVis"
              @l-drawing="drawing"
              @l-mousemove="hover"
              @l-click="click"
            ></v-canvas-layer>
          </v-map>
          </v-col>
        </v-row>
      </v-container>
    </v-content>

    <!-- <v-navigation-drawer v-model="right" fixed right temporary></v-navigation-drawer> -->

    <v-footer app color="grey" class="white--text">
      <span>Vuetify</span>
      <div class="flex-grow-1"></div>
      <span>&copy; 2019</span>
    </v-footer>
  </v-app>
</template>
  
<script>
import Revenue from "../components/Revenue.vue";
import * as Vue2Leaflet from "vue2-leaflet";
import { latLng, Icon, icon } from "leaflet";
import iconUrl from "leaflet/dist/images/marker-icon.png";
import shadowUrl from "leaflet/dist/images/marker-shadow.png";
import http from "../api";
import Vue2LeafletCanvas from "@skinnyjames/vue2-leaflet-canvas";


export default {
  components: {
    revenue: Revenue,
    "v-map": Vue2Leaflet.LMap,
    "v-tilelayer": Vue2Leaflet.LTileLayer,
    "l-marker": Vue2Leaflet.LMarker,
    "v-rev-marker": Vue2Leaflet.LMarker,
    "v-control-layers": Vue2Leaflet.LControlLayers,
    "v-canvas-layer": Vue2LeafletCanvas
  },
  props: {
    source: String
  },
  data: () => ({
    drawer: null,
    drawerRight: null,
    right: false,
    left: false,
    initialLocation: latLng(-29.0697, 24.6664),
    markerLatLng: [-29.0697, 24.6664],
    revLayerVis: false,
    siteVis: null,
    m_sm: 12,
    curSelSite: [{ propID: null }],
    rev_data: [],
    locations: [],
    geoj : null,
    revenue_year: 2017,
    revenue_month: 7,
    whatToShow: "Nothing",
    componentKey: 1,
    opacity: 0.8,
    zoom: 6,
    map: null,
    ctx: null,
    provider: {
      // This is the CanvasRenderingContext that children will draw to.
      context: null
    },
    canvas: null,
    imageUrls: [
      "Layer0_Symbol_b600e40_0.png",
      "Layer1_Symbol_b600bf0_0.png",
      "Layer2_Symbol_b60cc10_0.png",
      "Layer3_Symbol_b604e50_0.png",
      "Layer4_Symbol_b5fcbe0_0.png",
      "MTN_Layer0_Symbol_b600e40_0.png",
      "MTN_Layer1_Symbol_b600bf0_0.png",
      "MTN_Layer2_Symbol_b60cc10_0.png",
      "MTN_Layer3_Symbol_b604e50_0.png",
      "MTN_Layer4_Symbol_b5fcbe0_0.png",
      "VoiceVsData_sml3.png"
    ],
    imgs: [],
    imagesOK: 0,
    tileProviders: [
      {
        name: "OpenStreetMap",
        visible: true,
        attribution:
          '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
        url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
      },
      {
        name: "OpenTopoMap",
        visible: false,
        url: "https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png",
        attribution:
          'Map data: &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)'
      },
      {
        name: "Esri World Imagery",
        visible: false,
        url:
          "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
        attribution:
          "Tiles &copy; Esri &mdash; Source: Esri, i-cubed, USDA, USGS, AEX, GeoEye, Getmapping, Aerogrid, IGN, IGP, UPR-EGP, and the GIS User Community"
      }
    ]
  }),

  methods: {
    toggleRevenueLayer(e) {
      this.revLayerVis = e[0];
      var t = e[1].split("-");
      this.revenue_year = t[0];
      this.revenue_month = t[1];
      this.whatToShow = e[2];
      this.opacity = e[3] / 100;
      console.log(e);
      // eslint-disable-next-line no-console
      console.log("parent notified, changing Revenue Layer" + e);
      if (e[0]) {
        http
          .get(
            "http://10.220.67.70:8091/api/revenueDetail/?year=" +
              this.revenue_year +
              "&month=" +
              this.revenue_month
          )
          .then(response => {
            this.rev_data = response.data;
            //this.geoj = GeoJSON.parse(this.rev_data, {Point: ['latitude', 'longitude']});
            // eslint-disable-next-line no-console
            this.rev_data.forEach(element => {
              element.latlng = latLng(element.latitude, element.longitude);
            });
            this.locations = this.rev_data;
            //console.log(this.rev_data);
          })
          .catch(e => {
            // eslint-disable-next-line no-console
            console.log(e);
          });
      }
      this.redrawCanvas();
    },
    loadAllImages() {
      // eslint-disable-next-line no-console
      //console.log("Loading Images");
      for (var i = 0; i < this.imageUrls.length; i++) {
        var img = new Image();
        this.imgs.push(img);
        img.onload = function() {
          this.imagesOK++;
          if (this.imagesOK >= 9) {
            // eslint-disable-next-line no-console
            console.log("all images loaded");
          }
        };
        img.onerror = function() {
          alert("image load failed");
        };
        img.crossOrigin = "anonymous";
        img.src = this.imageUrls[i];
      }
    },
    drawing(info) {
      let canvas = info.canvas;
      let ctx = canvas.getContext("2d");
      // eslint-disable-next-line no-console
      //console.log("Zoom : "+this.zoom);
      ctx.globalAlpha = this.opacity;
      let map = this.$refs.map.mapObject;
      let bounds = map.getBounds();
      var f = 8;
      // eslint-disable-next-line no-console
      console.log("drawing");
      ctx.drawImage(this.imgs[10], 0, 0, 150, 405);
      for (let i = 0; i < this.rev_data.length; i++) {
        if (bounds.contains(this.rev_data[i].latlng)) {
          var dot = map.latLngToContainerPoint(this.rev_data[i].latlng);
          var min = 0;
          var max = 0;
          var x = 0;
          var r = 0;
          var voice =
            this.rev_data[i].sumMtcValue + this.rev_data[i].sumMocValue;
          var sgsn = this.rev_data[i].sumSgsnValue;
          if (this.rev_data[i].mno === "MTN") {
            switch (true) {
              case voice < 75001:
                x = 5;
                break;
              case voice < 150001:
                x = 6;
                break;
              case voice < 225001:
                x = 7;
                break;
              case voice < 305001:
                x = 8;
                break;
              case voice > 305000:
                x = 9;
                break;
            }
            switch (true) {
              case sgsn < 15001:
                r = 8 * (this.zoom / f);
                break;
              case sgsn < 30001:
                r = 12 * (this.zoom / f);
                break;
              case sgsn < 70001:
                r = 16 * (this.zoom / f);
                break;
              case sgsn < 100001:
                r = 20 * (this.zoom / f);
                break;
              case sgsn > 100000:
                r = 24 * (this.zoom / f);
                break;
            }
          } else {
            switch (true) {
              case voice < 75001:
                x = 0;
                break;
              case voice < 150001:
                x = 1;
                break;
              case voice < 225001:
                x = 2;
                break;
              case voice < 305001:
                x = 3;
                break;
              case voice > 305000:
                x = 4;
                break;
            }
            switch (true) {
              case sgsn < 15001:
                r = 8 * (this.zoom / f);
                break;
              case sgsn < 30001:
                r = 12 * (this.zoom / f);
                break;
              case sgsn < 70001:
                r = 16 * (this.zoom / f);
                break;
              case sgsn < 100001:
                r = 20 * (this.zoom / f);
                break;
              case sgsn > 100000:
                r = 24 * (this.zoom / f);
                break;
            }
          }
          if (
            this.rev_data[i].mno == "MTN" &&
            (this.whatToShow == "CCMTN" || this.whatToShow == "MTN")
          )
            ctx.drawImage(this.imgs[x], dot.x - r / 2, dot.y - r / 2, r, r);

          if (
            this.rev_data[i].mno == "CELLC" &&
            (this.whatToShow == "CCMTN" || this.whatToShow == "CC")
          )
            ctx.drawImage(this.imgs[x], dot.x - r / 2, dot.y - r / 2, r, r);
        }
      }
    },
    hover(info) {
      let vm = this;
      let container = document.getElementsByClassName("leaflet-container")[0];
      // isMoused has a boolean describing if a location is hovered over
      if (info.isMoused) {
        container.style.cursor = "crosshair";
      } else {
        container.style.cursor = "";
      }
    },
    click(info) {
      let vm = this;
      // clickedLocations will have any clicked location objects
      let points = info.clickedLocations;
      console.log(this.curSelSite);
      if (this.curSelSite == points[0] || this.curSelSite == null) {
        this.siteVis = !this.siteVis;
      }
     
      if (points[0]) {
        this.curSelSite = points[0];
      }
    },
    methodThatForcesUpdate() {
      this.$router.go();
      //this.componentKey += 1;
    },
    redrawCanvas() {
      this.componentKey += 1;
    }
  },
  provide() {
    return {
      provider: this.provider
    };
  },
  mounted() {
    this.loadAllImages();
    //this.provider.context = this.$refs['my-canvas'].getContext('2d');
    setTimeout(() => {
      // eslint-disable-next-line no-console
      console.log("done");
      this.$nextTick(() => {
        this.clusterOptions = { disableClusteringAtZoom: 11 };
      });
    }, 5000);
  }
};
</script>

<style>
@import "~leaflet/dist/leaflet.css";
#inspire > div > main > div > div > div > div {
  height: 900px !important;
}
#inspire > div > main > div > div > div > div:nth-child(1) {
  width: 100%;
}
.v-label {
  font-size: 0.828em;
}
</style>