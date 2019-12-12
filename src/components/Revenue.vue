<template>
  <div id="RevenueBox">
    <v-container fluid>
      <v-layout wrap>
        <v-switch
          app
          dense
          fixed
          :input-value="vis"
          @change="onChangeEventHandler1($event)"
        >Switch 1</v-switch>

        <div class="text-center">
          <v-menu offset-y>
            <template v-slot:activator="{ on }">
              <v-btn color="primary" dark v-on="on" dense>Revenue Months</v-btn>
            </template>
            <v-list dense>
              <v-list-item
                dense
                v-for="(item, index) in rev_months"
                :key="index"
                @click="onChangeDateHandler(item)"
              >
                <v-list-item-title>{{ item }}</v-list-item-title>
              </v-list-item>
            </v-list>
          </v-menu>
          <v-container class="grey lighten-5">
            <v-row>
              <v-col cols="12" sm="4">
                <v-slider v-model="opacity" vertical dense label="Opacity" @change="onChangeEventHandler4($event)" :value=80></v-slider>
              </v-col>
              <v-col cols="12" sm="4">
                <v-checkbox
                  v-model="checkbox1"
                  label="CellC"
                  @change="onChangeEventHandler2($event)"
                ></v-checkbox>
              </v-col>
              <v-col cols="12" sm="4">
                <v-checkbox v-model="checkbox2" label="MTN" @change="onChangeEventHandler3($event)"></v-checkbox>
              </v-col>
            </v-row>
          </v-container>
        </div>
        <div class="grey lighten-5 pa-4">
          <v-alert
            dense
            v-model="show"
            v-bind="localAttrs"
            :dismissible="dismissible"
            :type="type"
            class="mb-4"
            :icon="false"
          >Selected Revenue Month : {{selected_month}}</v-alert>
        </div>
      </v-layout>
    </v-container>
  </div>
</template>

<script>
import http from "../api";

export default {
  name: "revenue",
  components: {},
  data: function() {
    return {
      selected_month: "",
      rev_months: [],
      vis: false,
      show: true,
      type: "success",
      variant: null,
      border: "left",
      coloredBorder: false,
      dismissible: false,
      checkbox1: true,
      checkbox2: false,
      whatToShow: null,
      opacity: 80
    };
  },
  methods: {
    onChangeEventHandler1(event) {
      // eslint-disable-next-line no-console
      //console.log(event);
      this.vis = event;
      if (this.checkbox1 && this.checkbox2) {
        this.whatToShow = "CCMTN";
      } else if (this.checkbox1) {
        this.whatToShow = "CC";
      } else if (this.checkbox2) {
        this.whatToShow = "MTN";
      } else {
        this.whatToShow = "Nothing";

        // this.vis=false;
      }
      this.$emit("toggleRevenueLayer", [
        this.vis,
        this.selected_month,
        this.whatToShow,
        this.opacity
      ]);
    },
    onChangeEventHandler2(event) {
      // eslint-disable-next-line no-console
      //console.log(event);
      this.checkbox1 = event;
      if (this.checkbox1 && this.checkbox2) {
        this.whatToShow = "CCMTN";
      } else if (this.checkbox1) {
        this.whatToShow = "CC";
      } else if (this.checkbox2) {
        this.whatToShow = "MTN";
      } else {
        this.whatToShow = "Nothing";
        //this.vis=false;
      }
      this.$emit("toggleRevenueLayer", [
        this.vis,
        this.selected_month,
        this.whatToShow,
        this.opacity
      ]);
    },
    onChangeEventHandler3(event) {
      // eslint-disable-next-line no-console
      //console.log(event);
      this.checkbox2 = event;
      if (this.checkbox1 && this.checkbox2) {
        this.whatToShow = "CCMTN";
      } else if (this.checkbox1) {
        this.whatToShow = "CC";
      } else if (this.checkbox2) {
        this.whatToShow = "MTN";
      } else {
        this.whatToShow = "Nothing";
      }
      this.$emit("toggleRevenueLayer", [
        this.vis,
        this.selected_month,
        this.whatToShow,
        this.opacity
      ]);
    },
    onChangeEventHandler4(event) {
      // eslint-disable-next-line no-console
      console.log(this.opacity);
      //this.checkbox2 = event;
      if (this.checkbox1 && this.checkbox2) {
        this.whatToShow = "CCMTN";
      } else if (this.checkbox1) {
        this.whatToShow = "CC";
      } else if (this.checkbox2) {
        this.whatToShow = "MTN";
      } else {
        this.whatToShow = "Nothing";
      }
      this.$emit("toggleRevenueLayer", [
        this.vis,
        this.selected_month,
        this.whatToShow,
        this.opacity
      ]);
    },
    onChangeDateHandler(item) {
      // eslint-disable-next-line no-console
      console.log("REvenue Layer Date Change");
      this.selected_month = item;
      this.$emit("toggleRevenueLayer", [this.vis, this.selected_month,this.whatToShow,this.opacity]);
    }
  },
  computed: {
    localAttrs() {
      const attrs = {};

      if (this.variant === "border") {
        attrs.border = this.border;
        attrs.coloredBorder = this.coloredBorder;
      } else {
        attrs[this.variant] = true;
      }

      return attrs;
    }
  },
  mounted() {
    http
      .get("http://10.220.67.70:8091/api/findAllRevReports")
      .then(response => {
        // eslint-disable-next-line no-console
        //console.log(response.data);
        this.rev_months = response.data;
        this.selected_month = this.rev_months[0];
      })
      .catch(e => {
        // eslint-disable-next-line no-console
        console.log(e);
      });
  }
};
</script>
<style scoped>
.v-btn.v-size--default {
  font-size: 0.675rem;
}
.v-alert {
  font-size: 0.675rem;
}
.v-application .pa-4 {
  padding: 0px !important;
}
</style>
