<template>
  <app-btn
    v-if="paramList.length === 0 || !enableParams"
    :disabled="macro.disabledWhilePrinting && printerPrinting"
    @click="$emit('click', macro.name)"
    :style="borderStyle"
  >
    <slot></slot>
  </app-btn>
  <app-btn-group
    v-else
    :elevation="6"
  >
    <app-btn
      :disabled="macro.disabledWhilePrinting && printerPrinting"
      @click="$emit('click', macro.name)"
      :style="borderStyle"
    >
      <slot></slot>
    </app-btn>
    <v-menu
      left
      offset-y
      transition="slide-y-transition"
      :close-on-content-click="false"
    >
      <template v-slot:activator="{ on, attrs, value }">
        <app-btn
          v-if="paramList.length > 0"
          v-on="on"
          v-bind="attrs"
          :min-width="24"
          class="px-0"
        >
          <v-icon small :class="{ 'rotate-180': value }">$chevronDown</v-icon>
        </app-btn>
      </template>
      <v-card>
        <v-card-text class="pb-3 px-3">

          <v-layout wrap style="max-width: 150px;">

            <v-text-field
              v-for="(param, i) in paramList"
              :key="param"
              :label="param"
              outlined
              dense
              hide-details="auto"
              v-model="params[param].value"
              class=""
              :class="{ 'mb-3': (i < paramList.length - 1) }">

            <template v-slot:append>
              <app-btn
                @click="params[param].value = params[param].reset"
                style="margin-top: -4px; margin-right: -6px;"
                color=""
                icon
                small
              >
                <v-icon small>$reset</v-icon>
              </app-btn>

            </template>
            </v-text-field>

          </v-layout>

        </v-card-text>
        <v-divider />
        <v-card-actions class="px-3 py-3">
          <app-btn
            block
            @click="$emit('click', runCommand)"
          >
            Send
          </app-btn>
        </v-card-actions>
      </v-card>
    </v-menu>
  </app-btn-group>
</template>

<script lang="ts">
import { Component, Prop, Mixins } from 'vue-property-decorator'
import StateMixin from '@/mixins/state'
import { Macro } from '@/store/macros/types'

@Component({})
export default class AppMacroBtn extends Mixins(StateMixin) {
  @Prop({ type: Object, required: true })
  macro!: Macro

  @Prop({ type: Boolean, default: false })
  enableParams!: boolean;

  params: { [index: string]: { value: string | number; reset: string | number }} = {}

  get paramList () {
    return Object.keys(this.params)
  }

  /**
   * The formatted run command for a macro.
   */
  get runCommand () {
    let s = this.macro.name
    if (this.params) {
      for (const param of Object.keys(this.params)) {
        s += ` ${param}=${this.params[param].value}`
      }
    }
    return s
  }

  get borderStyle () {
    if (this.macro && this.macro.color !== '') {
      return `border-color: ${this.macro.color} !important; border-left: solid 4px ${this.macro.color} !important;`
    }
    return ''
  }

  mounted () {
    if (!this.macro.config || !this.macro.config.gcode) return []
    if (this.macro.config.gcode) {
      const regex = /params\.(\w+).*\|\s*default\s*\(\s*([^,)]+)/gmi
      let match = regex.exec(this.macro.config.gcode)
      do {
        if (match && match[1]) {
          const name = match[1]
          const value = (match[2] || '').trim()
          if (!this.params[name]) {
            this.$set(this.params, name, { value, reset: value })
          }
        }
      } while (
        (match = regex.exec(this.macro.config.gcode)) !== null
      )
    }
  }
}
</script>

<style lang="scss" scoped>
  .macro-params {
    height: 160px;
    display: flex;
    flex-direction: column;
    flex-wrap: wrap;
  }
  .macro-params > * {
    flex: 1 1 40px;
  }
</style>
