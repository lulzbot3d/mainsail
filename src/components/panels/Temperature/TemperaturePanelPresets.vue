<template>
    <div style="height: 100%">
        <v-menu v-if="presets.length" :offset-y="true" left>
            <template #activator="{ on, attrs }">
                <v-btn
                    text
                    tile
                    color="primary"
                    v-bind="attrs"
                    :disabled="['printing', 'paused'].includes(printer_state)"
                    class="pa-1"
                    v-on="on">
                    <span class="d-none ml-1 d-md-block">{{ $t('Panels.TemperaturePanel.Presets') }}</span>
                    <v-icon class="d-md-none">{{ mdiFire }}</v-icon>
                    <v-icon>{{ mdiMenuDown }}</v-icon>
                </v-btn>
            </template>
            <v-list dense class="py-0">
                <v-list-item v-for="(preset, index) of presets" :key="index" link @click="preheat(preset)">
                    <div class="d-flex align-center _preset-title">
                        <v-icon small class="mr-1">{{ mdiFire }}</v-icon>
                        <span style="padding-top: 2px">{{ preset.name }}</span>
                    </div>
                </v-list-item>
            </v-list>
            <v-divider class="_fix_transparency" />
            <v-list dense class="py-0">
                <v-list-item link @click="cooldown">
                    <div class="d-flex align-center _preset-title">
                        <v-icon small color="primary" class="mr-1">{{ mdiSnowflake }}</v-icon>
                        <span class="primary--text">{{ $t('Panels.TemperaturePanel.Cooldown') }}</span>
                    </div>
                </v-list-item>
            </v-list>
        </v-menu>
        <v-btn
            v-else
            :icon="$vuetify.breakpoint.smAndDown"
            :text="$vuetify.breakpoint.mdAndUp"
            :disabled="['printing', 'paused'].includes(printer_state)"
            tile
            @click="cooldown">
            <v-icon small>{{ mdiSnowflake }}</v-icon>
            <span class="d-none ml-1 d-md-inline">{{ $t('Panels.TemperaturePanel.Cooldown') }}</span>
        </v-btn>
        <v-btn
            v-if="true"
            :icon="$vuetify.breakpoint.smAndDown"
            :text="$vuetify.breakpoint.mdAndUp"
            :disabled="['printing', 'paused'].includes(printer_state)"
            tile
            color="primary"
            @click="doSend('SET_HEATER_TEMPERATURE HEATER=extruder TARGET=180 \n SET_HEATER_TEMPERATURE HEATER=heater_bed TARGET=60')">
            <v-icon small>{{ mdiFire }}</v-icon>
            <span class="d-none ml-1 d-md-inline">{{ $t('Panels.TemperaturePanel.Preheat') }}</span>
        </v-btn>
    </div>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import { GuiPresetsStatePreset } from '@/store/gui/presets/types'
import { mdiFire, mdiMenuDown, mdiSnowflake } from '@mdi/js'

@Component
export default class TemperaturePanelPresets extends Mixins(BaseMixin) {
    mdiFire = mdiFire
    mdiMenuDown = mdiMenuDown
    mdiSnowflake = mdiSnowflake

    get presets(): GuiPresetsStatePreset[] {
        return this.$store.getters['gui/presets/getPresets'] ?? []
    }

    get cooldownGcode(): string {
        return this.$store.getters['gui/presets/getCooldownGcode']
    }

    preheat(preset: GuiPresetsStatePreset): void {
        for (const [name, attributes] of Object.entries(preset.values)) {
            if (attributes.bool) {
                const splits = name.split(' ')
                const printerObject = splits[0]
                let printerObjectName = splits[1] ?? splits[0]

                // set default heater command
                let command = 'SET_HEATER_TEMPERATURE'
                let commandAttribute = 'HEATER'

                // override command for temperature_fan
                if (printerObject === 'temperature_fan') {
                    command = 'SET_TEMPERATURE_FAN_TARGET'
                    commandAttribute = 'TEMPERATURE_FAN'
                }

                // build gcode
                const gcode = `${command} ${commandAttribute}=${printerObjectName} TARGET=${attributes.value}`

                this.$store.dispatch('server/addEvent', { message: gcode, type: 'command' })
                this.$socket.emit('printer.gcode.script', { script: gcode })
            }
        }

        if (preset.gcode !== '') {
            setTimeout(() => {
                this.$store.dispatch('server/addEvent', { message: preset.gcode, type: 'command' })
                this.$socket.emit('printer.gcode.script', { script: preset.gcode })
            }, 100)
        }
    }

    cooldown(): void {
        this.$store.dispatch('server/addEvent', { message: this.cooldownGcode, type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: this.cooldownGcode })
    }

    doSend(gcode: string) {
        this.$store.dispatch('server/addEvent', { message: gcode, type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: gcode })
    }
}
</script>

<style scoped>
._preset-title {
    font-size: 0.8125rem;
    font-weight: 500;
}

/*
workaround for fixing a transparency issue
which is assumingly a vuetify bug
*/
._fix_transparency {
    background-color: #1e1e1e;
}
</style>
