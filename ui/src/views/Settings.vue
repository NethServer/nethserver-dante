<template>
  <div>
    <h2>{{$t('settings.title')}}</h2>
    <div v-if="!view.isLoaded" class="spinner spinner-lg"></div>
    <div v-if="view.isLoaded">
      <h3>{{$t('settings.mining_configuration')}}</h3>
      <form class="form-horizontal" v-on:submit.prevent="miningNow()">
        <div v-if="isEmptyMiners" class="form-group">
          <div class="col-sm-2"></div>
          <div class="col-sm-5">
            <div class="alert alert-info no-mg-bottom">
              <span class="pficon pficon-info"></span>
              {{$t('settings.missing_miners')}}
            </div>
          </div>
        </div>
        <div class="form-group">
          <label class="col-sm-2 control-label" for="textInput-modal-markup"></label>
          <div class="col-sm-5">
            <button class="btn btn-primary">{{$t('settings.mining_now')}}</button>
            <div
              v-if="view.isMining"
              class="spinner spinner-sm form-spinner-loader adjust-top-loader mg-left"
            ></div>
            <span v-if="view.miningSuccess" class="fa fa-check green mg-left"></span>
            <span v-if="view.miningFail" class="fa fa-times red mg-left"></span>
          </div>
        </div>
      </form>

      <h3>{{$t('settings.host_configuration')}}</h3>
      <form class="form-horizontal" v-on:submit.prevent="saveSettings('internet')">
        <div v-if="!isValidDns" class="form-group">
          <div class="col-sm-2"></div>
          <div class="col-sm-5">
            <div class="alert alert-warning no-mg-bottom">
              <span class="pficon pficon-warning-triangle-o"></span>
              {{$t('settings.invalid_dns')}}
            </div>
          </div>
        </div>
        <div :class="['form-group', errors.PublicHost.hasError ? 'has-error' : '']">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.public_host')}}</label>
          <div class="col-sm-5">
            <input class="form-control" v-model="settings.PublicHost">
            <span v-if="errors.PublicHost.hasError" class="help-block">
              {{$t('validation.validation_failed')}}:
              {{$t('validation.'+errors.PublicHost.message)}}
            </span>
          </div>
        </div>
        <div class="form-group">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.full_url')}}</label>
          <div class="col-sm-5">
            <input
              disabled
              class="form-control"
              :value="'https://'+settings.PublicHost+'/'+settings.alias+'/#/?theme='+settings.Theme+'&palette='+settings.Palette+'&last='+settings.Interval+'&lang='+settings.Language"
            >
          </div>
          <div class="col-sm-2">
            <a
              class="btn btn-primary"
              :href="'https://'+settings.PublicHost+'/'+settings.alias+'/#/?theme='+settings.Theme+'&palette='+settings.Palette+'&last='+settings.Interval+'&lang='+settings.Language"
              target="_blank"
            >{{$t('settings.open')}}</a>
          </div>
        </div>
      </form>
      <h3>{{$t('settings.report_configuration')}}</h3>
      <form class="form-horizontal" v-on:submit.prevent="saveSettings('internet')">
        <div :class="['form-group', errors.Theme.hasError ? 'has-error' : '']">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.theme')}}</label>
          <div class="col-sm-5">
            <select v-model="settings.Theme" class="form-control">
              <option value="light">{{$t('settings.light')}}</option>
              <option value="dark">{{$t('settings.dark')}}</option>
            </select>
            <span v-if="errors.Theme.hasError" class="help-block">
              {{$t('validation.validation_failed')}}:
              {{$t('validation.'+errors.Theme.message)}}
            </span>
          </div>
        </div>
        <div :class="['form-group', errors.Palette.hasError ? 'has-error' : '']">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.palette')}}</label>
          <div class="col-sm-5">
            <select v-model="settings.Palette" class="form-control">
              <option
                v-for="(p,pk) in palettes"
                :key="pk"
                :value="p.name"
              >{{$t('settings.palette')}} {{p.name.charAt(7)}}</option>
            </select>
            <span v-if="errors.Palette.hasError" class="help-block">
              {{$t('validation.validation_failed')}}:
              {{$t('validation.'+errors.Palette.message)}}
            </span>
          </div>
        </div>
        <div class="form-group">
          <label class="col-sm-2 control-label"></label>
          <div class="col-sm-5">
            <span
              class="palette"
              :style="'background: '+m"
              v-for="(m,mk) in paletteMap[settings.Palette]"
              :key="mk"
            >{{m}}</span>
          </div>
        </div>
        <div :class="['form-group', errors.Interval.hasError ? 'has-error' : '']">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.interval')}}</label>
          <div class="col-sm-5">
            <select v-model="settings.Interval" class="form-control">
              <option value="week">{{$t('settings.week')}}</option>
              <option value="month">{{$t('settings.month')}}</option>
              <option value="halfyear">{{$t('settings.halfyear')}}</option>
            </select>
            <span v-if="errors.Interval.hasError" class="help-block">
              {{$t('validation.validation_failed')}}:
              {{$t('validation.'+errors.Interval.message)}}
            </span>
          </div>
        </div>
        <div :class="['form-group', errors.Language.hasError ? 'has-error' : '']">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.language')}}</label>
          <div class="col-sm-5">
            <select :disabled="!languages" v-model="settings.Language" class="form-control">
              <option v-for="(l,lk) in languages" :key="lk" :value="l">{{l}}</option>
            </select>
            <span v-if="errors.Language.hasError" class="help-block">
              {{$t('validation.validation_failed')}}:
              {{$t('validation.'+errors.Language.message)}}
            </span>
          </div>
        </div>
        <div :class="['form-group', errors.Anonymization.hasError ? 'has-error' : '']">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.anonymization')}}</label>
          <div class="col-sm-5">
            <input class="form-control" type="checkbox" v-model="settings.Anonymization">
            <span v-if="errors.Anonymization.hasError" class="help-block">
              {{$t('validation.validation_failed')}}:
              {{$t('validation.'+errors.Anonymization.message)}}
            </span>
          </div>
        </div>
        <div :class="['form-group', errors.MaxEntries.hasError ? 'has-error' : '']">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.max_entries')}}</label>
          <div class="col-sm-5">
            <input class="form-control" type="number" v-model="settings.MaxEntries">
            <span v-if="errors.MaxEntries.hasError" class="help-block">
              {{$t('validation.validation_failed')}}:
              {{$t('validation.'+errors.MaxEntries.message)}}
            </span>
          </div>
        </div>

        <div
          v-for="(a,ak) in addresses"
          :key="ak"
          :class="['form-group', errors.Language.hasError ? 'has-error' : '']"
        >
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{ak == 0 ? $t('settings.addresses') : ''}}</label>
          <div class="col-sm-3">
            <input required type="email" class="form-control" v-model="a.address">
          </div>
          <div class="col-sm-2">
            <select v-model="a.day" class="form-control">
              <option value="every">{{$t('settings.every_day')}}</option>
              <option value="1">{{$t('settings.monday')}}</option>
              <option value="2">{{$t('settings.tuesday')}}</option>
              <option value="3">{{$t('settings.wednesday')}}</option>
              <option value="4">{{$t('settings.thursday')}}</option>
              <option value="5">{{$t('settings.friday')}}</option>
              <option value="6">{{$t('settings.saturday')}}</option>
              <option value="0">{{$t('settings.sunday')}}</option>
            </select>
          </div>
          <div class="col-sm-2">
            <select v-model="a.time" class="form-control">
              <option v-for="(t,tk) in times" :key="tk" :value="t.value">{{t.display}}</option>
            </select>
          </div>

          <div class="col-sm-1">
            <button type="button" @click="removeAddress(ak)" class="btn btn-default">
              <span class="fa fa-minus"></span>
            </button>
          </div>
          <div v-if="ak == addresses.length-1" class="col-sm-1">
            <button type="button" @click="addAddress()" class="btn btn-default">
              <span class="fa fa-plus"></span>
            </button>
          </div>
        </div>

        <div v-if="addresses.length == 0" class="form-group">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.addresses')}}</label>
          <div class="col-sm-1">
            <button type="button" @click="addAddress()" class="btn btn-default">
              <span class="fa fa-plus"></span>
            </button>
          </div>
        </div>

        <div class="form-group">
          <label class="col-sm-2 control-label" for="textInput-modal-markup">
            <div
              v-if="view.isSaving"
              class="spinner spinner-sm form-spinner-loader adjust-top-loader"
            ></div>
          </label>
          <div class="col-sm-5">
            <button class="btn btn-primary" type="submit">{{$t('settings.save')}}</button>
          </div>
        </div>
      </form>

      <h3>{{$t('settings.test_mail')}}</h3>
      <form class="form-horizontal" v-on:submit.prevent="testMail()">
        <div class="form-group">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.test_address')}}</label>
          <div class="col-sm-5">
            <input class="form-control" type="email" v-model="testSettings.address">
          </div>
        </div>
        <div class="form-group">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.include_configured_addresses')}}</label>
          <div class="col-sm-5">
            <input class="form-control" type="checkbox" v-model="testSettings.include">
          </div>
        </div>
        <div class="form-group">
          <label
            class="col-sm-2 control-label"
            for="textInput-modal-markup"
          >{{$t('settings.send_mail')}}</label>
          <div class="col-sm-2">
            <button type="submit" class="btn btn-primary">{{$t('settings.send')}}</button>
            <div
              v-if="view.isSending"
              class="spinner spinner-sm form-spinner-loader adjust-top-loader mg-left"
            ></div>
            <span v-if="view.sentSuccess" class="fa fa-check green mg-left"></span>
            <span v-if="view.sentFail" class="fa fa-times red mg-left"></span>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
var moment = require("moment");
export default {
  name: "Settings",
  mounted() {
    this.getSettings();
    this.getLanguages();
  },
  data() {
    return {
      view: {
        isLoaded: true,
        isSaving: false,
        isSending: false,
        sentSuccess: false,
        sentFail: false,
        isMining: false,
        miningSuccess: false,
        miningFail: false
      },
      testSettings: {
        address: "",
        include: false
      },
      isValidDns: true,
      isEmptyMiners: false,
      settings: {
        Theme: "dark",
        Palette: "palette1",
        Interval: "week",
        Language: "en",
        MailDestinations: [],
        PublicHost: "",
        Anonymization: false,
        MaxEntries: 10
      },
      addresses: [],
      times: [...Array(24).keys()].splice(1, 24).map(function(h) {
        return {
          display: moment((h - 1) * 3600 * 1000).format("HH:mm"),
          value: h
        };
      }),
      palettes: [
        {
          name: "palette1"
        },
        {
          name: "palette2"
        },
        {
          name: "palette3"
        },
        {
          name: "palette4"
        },
        {
          name: "palette5"
        },
        {
          name: "palette6"
        },
        {
          name: "palette7"
        },
        {
          name: "palette8"
        },
        {
          name: "palette9"
        }
      ],
      paletteMap: {
        palette1: ["#008FFB", "#00E396", "#FEB019", "#FF4560", "#775DD0"],
        palette2: ["#3F51B5", "#03A9F4", "#4CAF50", "#F9CE1D", "#FF9800"],
        palette3: ["#33B2DF", "#546E7A", "#D4526E", "#13D8AA", "#A5978B"],
        palette4: ["#4ECDC4", "#C7F464", "#81D4FA", "#546E7A", "#FD6A6A"],
        palette5: ["#2B908F", "#F9A3A4", "#90EE7E", "#FA4443", "#69D2E7"],
        palette6: ["#449DD1", "#F86624", "#EA3546", "#662E9B", "#C5D86D"],
        palette7: ["#D7263D", "#1B998B", "#2E294E", "#F46036", "#E2C044"],
        palette8: ["#662E9B", "#F86624", "#F9C80E", "#EA3546", "#43BCCD"],
        palette9: ["#5C4742", "#A5978B", "#8D5B4C", "#5A2A27", "#C4BBAF"]
      },
      languages: null,
      errors: this.initErrors()
    };
  },
  methods: {
    initErrors() {
      return {
        Theme: {
          hasError: false,
          message: ""
        },
        Palette: {
          hasError: false,
          message: ""
        },
        Interval: {
          hasError: false,
          message: ""
        },
        Language: {
          hasError: false,
          message: ""
        },
        MailDestinations: {
          hasError: false,
          message: ""
        },
        PublicHost: {
          hasError: false,
          message: ""
        },
        Anonymization: {
          hasError: false,
          message: ""
        },
        MaxEntries: {
          hasError: false,
          message: ""
        }
      };
    },
    addAddress() {
      this.addresses.push({
        address: "",
        day: 1,
        time: 1
      });
    },
    removeAddress(index) {
      this.addresses.splice(index, 1);
    },
    getSettings() {
      var context = this;

      context.view.isLoaded = false;
      nethserver.exec(
        ["nethserver-dante/settings/read"],
        {
          action: "configuration"
        },
        null,
        function(success) {
          try {
            success = JSON.parse(success);
          } catch (e) {
            console.error(e);
          }
          context.settings = success.settings;

          context.settings.Anonymization =
            context.settings.Anonymization == "enabled";
          context.settings.MaxEntries =
            parseInt(context.settings.MaxEntries) || 10;

          context.isValidDns = success.checks.valid;
          context.isEmptyMiners = success.checks.empty;

          for (var a in context.settings.MailDestinations) {
            var address = context.settings.MailDestinations[a];

            var parsedAddress = {
              address: address.mail,
              day:
                address.time.charAt(address.time.length - 1) == "*"
                  ? "every"
                  : address.time.charAt(address.time.length - 1),
              time: 
                address.time.charAt(3) == " "
                  ? address.time.charAt(2)
                  : address.time.substring(2,3)
                  
            };

            context.addresses.push(parsedAddress);
          }
          context.view.isLoaded = true;
        },
        function(error) {
          console.error(error);
        }
      );
    },
    getLanguages() {
      var context = this;

      nethserver.exec(
        ["nethserver-dante/settings/read"],
        {
          action: "languages"
        },
        null,
        function(success) {
          try {
            success = JSON.parse(success);
          } catch (e) {
            console.error(e);
          }
          context.languages = success.languages;
        },
        function(error) {
          console.error(error);
        }
      );
    },
    testMail() {
      var context = this;

      context.view.isSending = true;
      nethserver.exec(
        ["nethserver-dante/settings/execute"],
        {
          action: "test-mail",
          addresses: [context.testSettings.address].concat(
            context.testSettings.include
              ? context.settings.MailDestinations.map(function(m) {
                  return m.mail;
                })
              : []
          )
        },
        null,
        function(success) {
          context.view.isSending = false;
          context.view.sentSuccess = true;
          context.view.sentFail = false;

          setTimeout(function() {
            context.view.sentSuccess = false;
            context.view.sentFail = false;
          }, 3000);
        },
        function(error) {
          console.error(error);
          context.view.isSending = false;
          context.view.sentSuccess = false;
          context.view.sentFail = true;

          setTimeout(function() {
            context.view.sentSuccess = false;
            context.view.sentFail = false;
          }, 3000);
        }
      );
    },
    miningNow() {
      var context = this;

      context.view.isMining = true;
      nethserver.exec(
        ["nethserver-dante/settings/execute"],
        {
          action: "mining-now"
        },
        null,
        function(success) {
          context.view.isMining = false;
          context.view.miningSuccess = true;
          context.view.miningFail = false;

          setTimeout(function() {
            context.view.miningSuccess = false;
            context.view.miningFail = false;
          }, 3000);

          context.getSettings();
        },
        function(error) {
          console.error(error);
          context.view.isMining = false;
          context.view.miningSuccess = false;
          context.view.miningFail = true;

          setTimeout(function() {
            context.view.miningSuccess = false;
            context.view.miningFail = false;
          }, 3000);
        }
      );
    },
    saveSettings(type) {
      var context = this;
      var settingsObj = {
        action: "settings",
        PublicHost: context.settings.PublicHost,
        Theme: context.settings.Theme,
        Palette: context.settings.Palette,
        Interval: context.settings.Interval,
        Language: context.settings.Language,
        MailDestinations:
          context.addresses.length > 0
            ? context.addresses.map(function(a) {
                return {
                  time:
                    "0 " + a.time + " * * " + (a.day == "every" ? "*" : a.day),
                  mail: a.address
                };
              })
            : [],
        Anonymization: context.settings.Anonymization ? "enabled" : "disabled",
        MaxEntries: context.settings.MaxEntries
      };

      context.view.isSaving = true;
      context.errors = context.initErrors();
      nethserver.exec(
        ["nethserver-dante/settings/validate"],
        settingsObj,
        null,
        function(success) {
          context.view.isSaving = false;

          // notification
          nethserver.notifications.success = context.$i18n.t(
            "settings.settings_updated_ok"
          );
          nethserver.notifications.error = context.$i18n.t(
            "settings.settings_updated_error"
          );

          // update values
          nethserver.exec(
            ["nethserver-dante/settings/update"],
            settingsObj,
            function(stream) {
              console.info("settings-" + type, stream);
            },
            function(success) {
              context.addresses = [];
              context.getSettings();
            },
            function(error, data) {
              console.error(error, data);
            }
          );
        },
        function(error, data) {
          var errorData = {};
          context.view.isSaving = false;
          context.errors = context.initErrors();

          try {
            errorData = JSON.parse(data);
            for (var e in errorData.attributes) {
              var attr = errorData.attributes[e];
              context.errors[attr.parameter].hasError = true;
              context.errors[attr.parameter].message = attr.error;
            }
          } catch (e) {
            console.error(e);
          }
        }
      );
    }
  }
};
</script>

<style>
.adjust-top-loader {
  margin-top: -4px;
}

.palette {
  color: white;
  padding: 5px;
  margin: 5px;
}

.mg-left {
  margin-left: 15px;
}

.green {
  color: #3f9c35;
}

.red {
  color: #cc0000;
}

.no-mg-bottom {
  margin-bottom: 0px;
}
</style>
