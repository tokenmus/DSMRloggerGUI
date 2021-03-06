<template>
  <div class="settings">
    <v-container>
      <v-row text-center wrap justify="center">
        <v-col cols="12" sm="12" md="6">
          <v-row>
            <v-col cols="12" sm="12" class="d-flex justify-space-between">
              <h1>{{ $t("settings") }}</h1>
              <refresh-button dispatch="getSettings"></refresh-button>
            </v-col>
            <v-col cols="12" sm="12">
              <v-card class="mb-4" elevation="4" :loading="isLoading">
                <v-card-text>
                  <v-form ref="form" v-model="valid" lazy-validation>
                    <v-text-field
                      v-for="(value, name) in settingsData"
                      :key="name"
                      v-model="settingsData[name].value"
                      :label="$t(`settings_${settingsData[name].name}`)"
                      :rules="settingsData[name].rules || []"
                      :type="settingsData[name].type || 'text'"
                      :counter="
                        settingsData[name].maxlen
                          ? settingsData[name].maxlen
                          : null
                      "
                      filled
                      required
                      :loading="isLoading"
                    ></v-text-field>
                    <v-btn
                      class="ma-2"
                      :disabled="isLoading || !valid"
                      @click="saveSettings"
                    >
                      {{ $t("save") }}
                      <v-icon right>mdi-content-save</v-icon>
                    </v-btn>
                  </v-form>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import { mapState } from "vuex";
import RefreshButton from "@/components/RefreshButton.vue";
import mixin from "@/mixin";

export default {
  name: "settings",
  components: { RefreshButton },
  mixins: [mixin],
  data: () => ({
    valid: true
  }),
  computed: {
    settingsData() {
      try {
        // Add rules for each object.
        return this.settings.map(obj => {
          switch (obj.name) {
            case "ed_tariff1":
            case "ed_tariff2":
            case "er_tariff1":
            case "er_tariff2":
            case "gd_tariff":
              obj.type = "number";
              obj.rules = [
                v => /^\d+(\.\d{1,5})?$/.test(v) || this.$t("wrong_format"),
                v => v >= obj.min || this.$t("number_too_small"),
                v => v <= obj.max || this.$t("number_too_big")
              ];
              break;
            case "electr_netw_costs":
            case "gas_netw_costs":
              obj.type = "number";
              obj.rules = [
                v => /^\d+(\.\d{1,2})?$/.test(v) || this.$t("wrong_format"),
                v => v >= obj.min || this.$t("number_too_small"),
                v => v <= obj.max || this.$t("number_too_big")
              ];
              break;
            case "tlgrm_interval":
            case "mqtt_broker_port":
            case "mqtt_interval":
              obj.type = "number";
              obj.rules = [
                v => /^([+-]?[1-9]\d*|0)$/.test(v) || this.$t("wrong_format"),
                v => v >= obj.min || this.$t("number_too_small"),
                v => v <= obj.max || this.$t("number_too_big")
              ];
              break;
            case "mqtt_broker":
            case "mqtt_user":
            case "mqtt_passwd":
            case "mqtt_topTopic":
            case "mindergastoken":
              obj.type = "text";
              obj.rules = [v => v.length <= obj.maxlen || this.$t("too_long")];
          }

          return obj;
        });
      } catch (error) {
        return this.settings;
      }
    },
    ...mapState({
      isLoading: state => state.isLoading,
      settings: state => state.settings
    })
  },
  created() {
    if (!this.$store.state.settings.length) {
      this.$store.dispatch("getSettings");
    }
  },
  methods: {
    saveSettings() {
      // Remove unwanted properties from each object and return a promise, so we can execute them all at once.
      const data = this.settingsData.map(({ name, value }) => {
        return this.$store.dispatch("postSettings", {
          name,
          value
        });
      });

      // Set the loading indicator to true, so we can set it to false when every setting has been posted.
      this.$store.dispatch("setLoadingStatus", true);
      Promise.all(data).then(() => {
        this.$store.dispatch("getSettings");
      });
    }
  }
};
</script>
