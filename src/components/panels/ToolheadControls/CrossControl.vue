<style scoped>
.btnMinWidthAuto {
    min-width: auto !important;
}
</style>

<template>
    <responsive
        :breakpoints="{
            small: (el) => el.width <= 300,
        }">
        <template #default="{ el }">
            <div>
                <v-row>
                    <!-- DIRECTION BUTTONS -->
                    <v-col>
                        <v-row dense class="mb-1">
                            <v-col cols="3"></v-col>
                            <v-col cols="3"> <!-- Y Up /*:color="homedAxes.includes('y') ? 'success' : 'warning'"*/-->
                                <v-btn
                                    class="btnMinWidthAuto fill-width"
                                    :disabled="
                                        !yAxisHomed ||
                                        selectedCrossStep === null ||
                                        selectedCrossStep === undefined ||
                                        ['printing'].includes(printer_state)
                                    "
                                    :color="homedAxes.includes('y') ? '#7b7b7b' : ''"
                                    @click="doSendMove(`Y${reverseY ? '-' : '+'}${stepSize}`, feedrateXY)">
                                    <v-icon>{{ mdiChevronUp }}</v-icon>
                                </v-btn>
                            </v-col>
                            <v-col cols="3"></v-col>
                            <v-col cols="3"> <!-- Z Up -->
                                <v-btn
                                    class="btnMinWidthAuto fill-width"
                                    :disabled="
                                        !zAxisHomed ||
                                        selectedCrossStep === null ||
                                        selectedCrossStep === undefined ||
                                        ['printing'].includes(printer_state)
                                    "
                                    :color="homedAxes.includes('z') ? '#7b7b7b' : ''"
                                    @click="doSendMove(`Z${reverseZ ? '-' : '+'}${stepSize}`, feedrateZ)">
                                    <v-icon>{{ mdiChevronUp }}</v-icon>
                                </v-btn>
                            </v-col>
                        </v-row>
                        <v-row dense>
                            <v-col cols="3" class="p-rel"> <!-- X left -->
                                <v-btn
                                    class="btnMinWidthAuto fill-width p-abs"
                                    style="top: -50%; width: calc(100% - 8px)"
                                    :disabled="
                                        !xAxisHomed ||
                                        selectedCrossStep === null ||
                                        selectedCrossStep === undefined ||
                                        ['printing'].includes(printer_state)
                                    "
                                    :color="homedAxes.includes('x') ? '#7b7b7b' : ''"
                                    @click="doSendMove(`X${!reverseX ? '-' : '+'}${stepSize}`, feedrateXY)">
                                    <v-icon>{{ mdiChevronLeft }}</v-icon>
                                </v-btn>
                            </v-col>
                            <v-col cols="3"> <!-- Y Down -->
                                <v-btn
                                    class="btnMinWidthAuto fill-width"
                                    :disabled="
                                        !yAxisHomed ||
                                        selectedCrossStep === null ||
                                        selectedCrossStep === undefined ||
                                        ['printing'].includes(printer_state)
                                    "
                                    :color="homedAxes.includes('y') ? '#7b7b7b' : ''"
                                    @click="doSendMove(`Y${!reverseY ? '-' : '+'}${stepSize}`, feedrateXY)">
                                    <v-icon>{{ mdiChevronDown }}</v-icon>
                                </v-btn>
                            </v-col>
                            <v-col cols="3" class="p-rel"> <!-- X Right -->
                                <v-btn
                                    class="btnMinWidthAuto fill-width p-abs"
                                    style="top: -50%; width: calc(100% - 8px)"
                                    :disabled="
                                        !xAxisHomed ||
                                        selectedCrossStep === null ||
                                        selectedCrossStep === undefined ||
                                        ['printing'].includes(printer_state)
                                    "
                                    :color="homedAxes.includes('x') ? '#7b7b7b' : ''"
                                    @click="doSendMove(`X${reverseX ? '-' : '+'}${stepSize}`, feedrateXY)">
                                    <v-icon>{{ mdiChevronRight }}</v-icon>
                                </v-btn>
                            </v-col>
                            <v-col cols="3"> <!-- Z Down -->
                                <v-btn
                                    class="btnMinWidthAuto fill-width"
                                    :disabled="
                                        !zAxisHomed ||
                                        selectedCrossStep === null ||
                                        selectedCrossStep === undefined ||
                                        ['printing'].includes(printer_state)
                                    "
                                    :color="homedAxes.includes('z') ? '#7b7b7b' : ''"
                                    @click="doSendMove(`Z${!reverseZ ? '-' : '+'}${stepSize}`, feedrateZ)">
                                    <v-icon>{{ mdiChevronDown }}</v-icon>
                                </v-btn>
                            </v-col>
                        </v-row>
                    </v-col>
                    <!-- HOME / 5th ACTION BUTTONS -->
                    <v-col v-if="!el.is.small" class="d-flex align-center">
                        <div class="flex-grow-1" style="border-radius: 4px; overflow: hidden">
                            <v-row dense style="margin-bottom: -2px !important">
                                <v-col cols="6">
                                    <v-btn
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('homeAll')"
                                        :color="homedAxes.includes('xyz') ? '' : '#7b7b7b'"
                                        height="30"
                                        tile
                                        class="w-100"
                                        @click="doHome">
                                        <div class="d-flex align-center">
                                            <v-icon>{{ mdiHome }}</v-icon>
                                            <span>{{ $t('Panels.ToolheadControlPanel.ALL') }}</span>
                                        </div>
                                    </v-btn>
                                </v-col>
                                <v-col cols="6" class="d-flex">
                                    <v-btn
                                        v-if="actionButton === 'qgl'"
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('qgl')"
                                        :color="colorQuadGantryLevel"
                                        height="30"
                                        dense
                                        tile
                                        class="btnMinWidthAuto flex-grow-1 px-0"
                                        @click="doQGL">
                                        {{ $t('Panels.ToolheadControlPanel.QGL') }}
                                    </v-btn>
                                    <v-btn
                                        v-else-if="actionButton === 'ztilt'"
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('zTilt')"
                                        :color="colorZTilt"
                                        height="30"
                                        dense
                                        tile
                                        class="btnMinWidthAuto flex-grow-1 px-0"
                                        @click="doZtilt">
                                        {{ $t('Panels.ToolheadControlPanel.ZTilt') }}
                                    </v-btn>
                                    <v-btn
                                        v-else
                                        :disabled="['printing'].includes(printer_state)"
                                        :color="homedAxes == '' ? '' : '#7b7b7b'"
                                        height="30"
                                        dense
                                        tile
                                        class="flex-grow-1 px-0"
                                        @click="doSend('M84')">
                                        <v-icon>{{ mdiEngineOff }}</v-icon>
                                    </v-btn>
                                </v-col>
                            </v-row>
                            <!-- X/Y/Z HOME BUTTONS -->
                            <v-row dense>
                                <v-col v-if="!enableXYHoming" cols="4" class="flex-grow-1">
                                    <v-btn
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('homeX')"
                                        :color="homedAxes.includes('x') ? '' : '#7b7b7b'"
                                        tile
                                        height="30"
                                        class="btnMinWidthAuto w-100"
                                        @click="doHomeX">
                                        X
                                    </v-btn>
                                </v-col>
                                <v-col v-if="!enableXYHoming" cols="4" class="flex-grow-1">
                                    <v-btn
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('homeY')"
                                        :color="homedAxes.includes('y') ? '' : '#7b7b7b'"
                                        tile
                                        height="30"
                                        class="btnMinWidthAuto w-100"
                                        @click="doHomeY">
                                        Y
                                    </v-btn>
                                </v-col>
                                <v-col v-else cols="6" class="flex-grow-1">
                                    <v-btn
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('homeY')"
                                        :color="homedAxes.includes('xy') ? '' : '#7b7b7b'"
                                        tile
                                        height="30"
                                        class="btnMinWidthAuto w-100"
                                        @click="doHomeXY">
                                        XY
                                    </v-btn>
                                </v-col>
                                <v-col :class="enableXYHoming ? 'col-6' : 'col-4'" class="flex-grow-1">
                                    <v-btn
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('homeZ')"
                                        :color="homedAxes.includes('z') ? '' : '#7b7b7b'"
                                        tile
                                        height="30"
                                        class="btnMinWidthAuto w-100"
                                        @click="doHomeZ">
                                        Z
                                    </v-btn>
                                </v-col>
                            </v-row>
                        </div>
                    </v-col>
                </v-row>
                <v-row v-if="el.is.small">
                    <!-- HOME / 5th ACTION BUTTONS -->
                    <v-col class="d-flex align-center">
                        <div class="flex-grow-1" style="border-radius: 4px; overflow: hidden">
                            <v-row dense style="margin-bottom: -2px !important">
                                <v-col cols="6">
                                    <v-btn
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('homeAll')"
                                        :color="homedAxes.includes('xyz') ? 'secondary' : 'success'"
                                        height="30"
                                        tile
                                        class="w-100"
                                        @click="doHome">
                                        <div class="d-flex align-center">
                                            <v-icon>{{ mdiHome }}</v-icon>
                                            <span>{{ $t('Panels.ToolheadControlPanel.ALL') }}</span>
                                        </div>
                                    </v-btn>
                                </v-col>
                                <v-col cols="6" class="d-flex">
                                    <v-btn
                                        v-if="actionButton === 'qgl'"
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('qgl')"
                                        :color="colorQuadGantryLevel"
                                        height="30"
                                        dense
                                        tile
                                        class="btnMinWidthAuto flex-grow-1 px-0"
                                        @click="doQGL">
                                        {{ $t('Panels.ToolheadControlPanel.QGL') }}
                                    </v-btn>
                                    <v-btn
                                        v-else-if="actionButton === 'ztilt'"
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('zTilt')"
                                        :color="colorZTilt"
                                        height="30"
                                        dense
                                        tile
                                        class="btnMinWidthAuto flex-grow-1 px-0"
                                        @click="doZtilt">
                                        {{ $t('Panels.ToolheadControlPanel.ZTilt') }}
                                    </v-btn>
                                    <v-btn
                                        v-else
                                        :disabled="['printing'].includes(printer_state)"
                                        :color="homedAxes !== '' ? 'secondary' : 'success'"
                                        height="30"
                                        dense
                                        tile
                                        class="flex-grow-1 px-0"
                                        @click="doSend('M84')">
                                        <v-icon>{{ mdiEngineOff }}</v-icon>
                                    </v-btn>
                                </v-col>
                            </v-row>
                            <!-- X/Y/Z HOME BUTTONS -->
                            <v-row dense>
                                <v-col v-if="!enableXYHoming" cols="4" class="flex-grow-1">
                                    <v-btn
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('homeX')"
                                        :color="homedAxes.includes('x') ? 'secondary' : 'success'"
                                        tile
                                        height="30"
                                        class="btnMinWidthAuto w-100"
                                        @click="doHomeX">
                                        X
                                    </v-btn>
                                </v-col>
                                <v-col v-if="!enableXYHoming" cols="4" class="flex-grow-1">
                                    <v-btn
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('homeY')"
                                        :color="homedAxes.includes('y') ? 'secondary' : 'success'"
                                        tile
                                        height="30"
                                        class="btnMinWidthAuto w-100"
                                        @click="doHomeY">
                                        Y
                                    </v-btn>
                                </v-col>
                                <v-col v-else cols="6" class="flex-grow-1">
                                    <v-btn
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('homeY')"
                                        :color="homedAxes.includes('xy') ? 'secondary' : '#59d313'"
                                        tile
                                        height="30"
                                        class="btnMinWidthAuto w-100"
                                        @click="doHomeXY">
                                        XY
                                    </v-btn>
                                </v-col>
                                <v-col :class="enableXYHoming ? 'col-6' : 'col-4'" class="flex-grow-1">
                                    <v-btn
                                        :disabled="['printing'].includes(printer_state)"
                                        :loading="loadings.includes('homeZ')"
                                        :color="homedAxes.includes('z') ? 'secondary' : '#59d313'"
                                        tile
                                        height="30"
                                        class="btnMinWidthAuto w-100"
                                        @click="doHomeZ">
                                        Z
                                    </v-btn>
                                </v-col>
                            </v-row>
                        </div>
                    </v-col>
                </v-row>
                <!-- STEP SIZE BUTTON GROUP -->
                <v-row no-gutters class="mt-3">
                    <v-col>
                        <v-btn-toggle
                            v-if="stepsReversed.length > 0"
                            :key="`all-steps-${stepsReversed.join('_')}`"
                            v-model="selectedCrossStep"
                            dense
                            mandatory
                            style="flex-wrap: nowrap; width: 100%">
                            <v-btn
                                v-for="step of stepsReversed"
                                :key="`step-${step}`"
                                :disabled="['printing'].includes(printer_state)"
                                dense
                                class="btnMinWidthAuto flex-grow-1 px-0"
                                style="height: 28px">
                                <span class="body-2">{{ step }}</span>
                            </v-btn>
                        </v-btn-toggle>
                        <div v-else class="font-weight-bold warning rounded pa-2">
                            {{ $t('Panels.ToolheadControlPanel.PleaseConfigureSteps') }}
                            <br />
                            <router-link style="color: white" to="/settings/interface">
                                {{ $t('Panels.ToolheadControlPanel.SettingsInterfaceControl') }}
                            </router-link>
                        </div>
                    </v-col>
                </v-row>
            </div>
        </template>
    </responsive>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import ControlMixin from '@/components/mixins/control'
import Responsive from '@/components/ui/Responsive.vue'
import { mdiChevronUp, mdiChevronLeft, mdiChevronRight, mdiChevronDown, mdiEngineOff, mdiHome } from '@mdi/js'

@Component({
    components: { Responsive },
})
export default class CrossControl extends Mixins(BaseMixin, ControlMixin) {
    mdiChevronUp = mdiChevronUp
    mdiChevronLeft = mdiChevronLeft
    mdiChevronRight = mdiChevronRight
    mdiChevronDown = mdiChevronDown
    mdiEngineOff = mdiEngineOff
    mdiHome = mdiHome

    get actionButton(): string {
        return this.$store.state.gui.control.actionButton ?? this.defaultActionButton
    }

    /**
     * Step size selection
     */
    get selectedCrossStep() {
        return this.$store.state.gui.control.selectedCrossStep
    }

    set selectedCrossStep(newVal) {
        this.$store.dispatch('gui/saveSetting', { name: 'control.selectedCrossStep', value: newVal })
    }

    get stepSize(): number {
        return this.stepsReversed[this.selectedCrossStep]
    }

    /**
     * Axes reverse states
     */
    get reverseX() {
        return this.$store.state.gui.control.reverseX
    }

    get reverseY() {
        return this.$store.state.gui.control.reverseY
    }

    get reverseZ() {
        return this.$store.state.gui.control.reverseZ
    }

    get stepsAll() {
        return this.$store.state.gui.control?.stepsAll ?? []
    }

    get stepsReversed() {
        return Array.from(new Set([...(this.stepsAll ?? [])])).sort((a, b) => a - b)
    }
}
</script>
