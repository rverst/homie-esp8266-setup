<template>
  <span>
    <p>
      Configure the settings for your device.
    </p>

    <form @submit.prevent="sendDone">
      <label class="label" for="friendly_name">Friendly name</label>
      <p class="control">
        <input v-model.trim="name" class="input" type="text" placeholder="My device name" id="friendly_name" required />
        <span class="help">Required.</span>
      </p>

      <hr/>

      <p v-if="deviceInformation.settings.length === 0">No custom settings.</p>
      <template v-else>
        <p>
          <h3>Custom settings</h3>
          <template v-for="setting in deviceInformation.settings">
            <template v-if="setting.type === 'bool'">
              <p class="control">
                <label class="checkbox" :for="'custom_setting_' + setting.name">
                  <input type="checkbox" v-model="customSettings[setting.name]" :id="'custom_setting_' + setting.name" /> {{ setting.description }}
                </label>
                <span class="help" v-html="setting.required ? 'Required.' : 'Optional. Defaults to <span class=\'tag\'>' + setting.default + '</span>.'"></span>
              </p>
            </template>
            <template v-else>
              <label class="label" :for="'custom_setting_' + setting.name"><span class="icon is-small"><i :class="{ fa: true, 'fa-hashtag': setting.type === 'long' || setting.type === 'ulong' || setting.type === 'double', 'fa-font': setting.type === 'string' }"></i></span> {{ setting.type === 'double' ? '(float)' : '' }} {{ setting.type === 'ulong' ? '(unsigned)' : '' }} {{ setting.description }}</label>
              <p class="control">
                <input v-if="setting.type === 'ulong'" v-model.number="customSettings[setting.name]" class="input" type="number" step="1" min="0" max="4294967295" :id="'custom_setting_' + setting.name" />
                <input v-if="setting.type === 'long'" v-model.number="customSettings[setting.name]" class="input" type="number" step="1" min="-2147483648" max="2147483647" :id="'custom_setting_' + setting.name" />
                <input v-if="setting.type === 'double'" v-model.number="customSettings[setting.name]" class="input" type="number" step="any" :id="'custom_setting_' + setting.name" />
                <input v-if="setting.type === 'string'" v-model.trim="customSettings[setting.name]" class="input" type="text" :id="'custom_setting_' + setting.name" />
                <span class="help" v-html="setting.required ? 'Required.' : 'Optional. Defaults to <span class=\'tag\'>' + setting.default + '</span>.'"></span>
              </p>
            </template>
          </template>
        </p>
      </template>

      <p class="control">
        <button type="submit" :disabled="!formIsValid" class="button is-primary">Next</button>
      </p>
    </form>
  </span>
</template>

<script>
export default {
  data () {
    const customSettings = {}

    this.deviceInformation.settings.forEach((setting) => {
      customSettings[setting.name] = setting.required ? null : setting.default
    })
    return {
      name: null,
      customSettings
    }
  },
  props: ['deviceInformation'],
  computed: {
    formIsValid: function () {
      if (!this.name) return false

      for (let originalSetting of this.deviceInformation.settings) {
        if (originalSetting.required && this.customSettings[originalSetting.name] === '') return false
      }

      return true
    }
  },
  methods: {
    sendDone: function () {
      const settings = { settings: {} }
      settings.name = this.name

      for (let providedSettingName of Object.keys(this.customSettings)) {
        let originalSettingObject
        for (let settingObject of this.deviceInformation.settings) {
          if (settingObject.name === providedSettingName) {
            originalSettingObject = settingObject
            break
          }
        }

        if (originalSettingObject.required || originalSettingObject.default !== this.customSettings[providedSettingName]) {
          settings.settings[providedSettingName] = this.customSettings[providedSettingName]
        }
      }

      this.$emit('settingsConfig', settings)
      this.$emit('done')
    }
  }
}
</script>
