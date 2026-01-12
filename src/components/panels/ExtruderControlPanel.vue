<template>
    <panel
        v-if="showPanel"
        :icon="mdiPrinter3dNozzle"
        :title="$t('Panels.ExtruderControlPanel.Headline')"
        :collapsible="true"
        card-class="extruder-control-panel">
        <!-- PANEL-HEADER 3-DOT-MENU -->
        <template #buttons>
            <v-menu :offset-y="true" :close-on-content-click="false" left>
                <template #activator="{ on, attrs }">
                    <v-btn
                        small
                        color='#8b8b8b'
                        v-bind="attrs"
                        class="px-0"
                        style="min-width: 32px;"
                        v-on="on">
                        <v-icon />
                        <img height="40" src="@/assets/Meteor-01.svg" />
                        <v-icon />
                        <v-icon>{{ mdiMenuDown }}</v-icon>
                    </v-btn>
                </template>
                <v-list dense>
                    <v-list-item :disabled="printerIsPrintingOnly">
                        <v-btn small color="#7b7b7b" style="width: 100%" @click="doSend('MET175')">MET175</v-btn>
                    </v-list-item>
                    <v-list-item :disabled="printerIsPrintingOnly">
                        <v-btn small color="#7b7b7b" style="width: 100%" @click="doSend('MET285')">MET285</v-btn>
                    </v-list-item>
                </v-list>
            </v-menu>
            <v-menu v-if="showFilamentMacros" :offset-y="true" :close-on-content-click="false" left>
                <template #activator="{ on, attrs }">
                    <v-btn icon tile v-bind="attrs" v-on="on">
                        <v-icon>{{ mdiDotsVertical }}</v-icon>
                    </v-btn>
                </template>
                <v-list dense>
                    <!-- FILAMENT UNLOAD -->
                    <v-list-item v-if="unloadFilamentMacro">
                        <v-tooltip top :disabled="canExecuteUnloadMacro" color="secondary">
                            <template color='#7b7b7b' #activator="{ on }">
                                <div v-on="on">
                                    <macro-button
                                        :macro="unloadFilamentMacro"
                                        :alias="$t('Panels.ExtruderControlPanel.UnloadFilament')"
                                        :disabled="!canExecuteUnloadMacro || printerIsPrintingOnly"
                                        color="#7b7b7b" />
                                </div>
                            </template>
                            <span>
                                {{ $t('Panels.ExtruderControlPanel.ExtruderTempTooLow') }}
                                {{ minExtrudeTemp }} °C
                            </span>
                        </v-tooltip>
                    </v-list-item>
                    <!-- FILAMENT LOAD -->
                    <v-list-item v-if="loadFilamentMacro">
                        <v-tooltip top :disabled="canExecuteLoadMacro" color="secondary">
                            <template #activator="{ on }">
                                <div v-on="on">
                                    <macro-button
                                        :macro="loadFilamentMacro"
                                        :alias="$t('Panels.ExtruderControlPanel.LoadFilament')"
                                        :disabled="!canExecuteLoadMacro || printerIsPrintingOnly"
                                        color="#7b7b7b" />
                                </div>
                            </template>
                            <span>
                                {{ $t('Panels.ExtruderControlPanel.ExtruderTempTooLow') }}
                                {{ minExtrudeTemp }} °C
                            </span>
                        </v-tooltip>
                    </v-list-item>
                    <!-- FILAMENT PURGE -->
                    <v-list-item v-if="purgeFilamentMacro">
                        <v-tooltip top :disabled="canExecutePurgeMacro" color="primary">
                            <template #activator="{ on }">
                                <div v-on="on">
                                    <macro-button
                                        :macro="purgeFilamentMacro"
                                        :alias="$t('Panels.ExtruderControlPanel.PurgeFilament')"
                                        :disabled="!canExecutePurgeMacro || printerIsPrintingOnly"
                                        color="#7b7b7b" />
                                </div>
                            </template>
                            <span>
                                {{ $t('Panels.ExtruderControlPanel.ExtruderTempTooLow') }}
                                {{ minExtrudeTemp }} °C
                            </span>
                        </v-tooltip>
                    </v-list-item>
                    <!-- NOZZLE CLEAN -->
                    <v-list-item v-if="cleanNozzleMacro">
                        <macro-button
                            :macro="cleanNozzleMacro"
                            :alias="$t('Panels.ExtruderControlPanel.CleanNozzle')"
                            :disabled="printerIsPrintingOnly"
                            color="#7b7b7b" />
                    </v-list-item>
                </v-list>
            </v-menu>
            <extruder-panel-settings />
        </template>
        <!-- TOOL SELECTOR BUTTONS -->
        <extruder-control-panel-tools v-if="showTools && toolchangeMacros.length" />
        <!-- FIRMWARE RETRACTION SETTINGS -->
        <template v-if="showFirmwareRetraction">
            <v-divider v-if="showTools" />
            <firmware-retraction-settings />
        </template>
        <!-- EXTRUDER INPUTS AND QUICKSELECTS -->
        <template v-if="showExtruderControl">
            <v-divider v-if="showTools || showFirmwareRetraction" />
            <extruder-control-panel-control />
        </template>
        <!-- EXTRUSION FACTOR SLIDER -->
        <template v-if="showExtrusionFactor">
            <v-divider v-if="showTools || showExtrusionFactor" />
            <extrusion-factor-settings />
        </template>
        <!-- PRESSURE ADVANCE SETTINGS -->
        <template v-if="showPressureAdvance">
            <v-divider v-if="showTools || showPressureAdvance" />
            <extruder-pressure-advance-settings v-if="extruderSteppers.length === 0" />
            <template v-else>
                <extruder-stepper-pressure-advance-settings
                    v-for="(extruderStepper, index) in extruderSteppers"
                    :key="extruderStepper"
                    :class="{ 'pt-3': index === 0 }"
                    :extruder-stepper="extruderStepper" />
            </template>
        </template>
    </panel>
</template>

<script lang="ts">
import { mdiPrinter3dNozzle, mdiDotsVertical, mdiMenuDown } from '@mdi/js'
import { Component, Mixins } from 'vue-property-decorator'
import { PrinterStateMacro } from '@/store/printer/types'
import BaseMixin from '@/components/mixins/base'
import ControlMixin from '@/components/mixins/control'
import ExtruderMixin from '@/components/mixins/extruder'

@Component
export default class ExtruderControlPanel extends Mixins(BaseMixin, ControlMixin, ExtruderMixin) {
    mdiPrinter3dNozzle = mdiPrinter3dNozzle
    mdiDotsVertical = mdiDotsVertical
    mdiMenuDown = mdiMenuDown

    private heatWaitGcodes = ['printer.extruder.can_extrude', 'TEMPERATURE_WAIT', 'M109']

    get showPanel(): boolean {
        return this.klipperReadyForGui && this.extruders.length > 0
    }

    get macros() {
        return this.$store.getters['printer/getMacros']
    }

    get loadFilamentMacro(): PrinterStateMacro | undefined {
        const macros = ['LOAD_FILAMENT', 'FILAMENT_LOAD']

        return this.macros.find((macro: PrinterStateMacro) => macros.includes(macro.name.toUpperCase()))
    }

    get unloadFilamentMacro(): PrinterStateMacro | undefined {
        const macros = ['UNLOAD_FILAMENT', 'FILAMENT_UNLOAD']

        return this.macros.find((macro: PrinterStateMacro) => macros.includes(macro.name.toUpperCase()))
    }

    get purgeFilamentMacro(): PrinterStateMacro | undefined {
        const macros = ['PURGE_FILAMENT', 'FILAMENT_PURGE']

        return this.macros.find((macro: PrinterStateMacro) => macros.includes(macro.name.toUpperCase()))
    }

    get cleanNozzleMacro(): PrinterStateMacro | undefined {
        const macros = ['CLEAN_NOZZLE', 'NOZZLE_CLEAN', 'WIPE_NOZZLE', 'NOZZLE_WIPE']

        return this.macros.find((macro: PrinterStateMacro) => macros.includes(macro.name.toUpperCase()))
    }

    /**
     * test if the load and unload macro include specific keywords. if true, we allow
     * execution of that macro even if at the current time extrudePossible === false
     */
    get canExecuteLoadMacro(): boolean {
        if (this.extrudePossible) return true

        return this.heatWaitGcodes.some((gcode) => this.loadFilamentMacro?.prop.gcode.includes(gcode))
    }

    get canExecuteUnloadMacro(): boolean {
        if (this.extrudePossible) return true

        return this.heatWaitGcodes.some((gcode) => this.unloadFilamentMacro?.prop.gcode.includes(gcode))
    }

    get canExecutePurgeMacro(): boolean {
        if (this.extrudePossible) return true

        return this.heatWaitGcodes.some((gcode) => this.purgeFilamentMacro?.prop.gcode.includes(gcode))
    }

    get showFilamentMacros(): boolean {
        return (
            this.loadFilamentMacro !== undefined ||
            this.unloadFilamentMacro !== undefined ||
            this.purgeFilamentMacro !== undefined ||
            this.cleanNozzleMacro !== undefined
        )
    }

    get showTools(): boolean {
        if (this.toolchangeMacros.length < 1) return false

        return this.$store.state.gui.view.extruder.showTools ?? true
    }

    get showExtrusionFactor(): boolean {
        return this.$store.state.gui.view.extruder.showExtrusionFactor ?? true
    }

    get extruderSteppers() {
        return Object.keys(this.$store.state.printer)
            .filter((e) => e.startsWith('extruder_stepper '))
            .sort((a, b) => a.localeCompare(b))
    }

    get showPressureAdvance(): boolean {
        return this.$store.state.gui.view.extruder.showPressureAdvance ?? true
    }

    get showFirmwareRetraction(): boolean {
        if (!this.existsFirmwareRetraction) return false

        return this.$store.state.gui.view.extruder.showFirmwareRetraction ?? true
    }

    get showExtruderControl(): boolean {
        return this.$store.state.gui.view.extruder.showExtruderControl ?? true
    }
}
</script>
