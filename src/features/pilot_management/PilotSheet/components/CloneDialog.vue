<template>
  <cc-solo-dialog ref="dialog" icon="mdi-dna" large no-confirm title="Clone Pilot">
    <v-card-text>
      <v-row>
        <v-col>
          <div class="text-center body-text light-panel pa-2" style="border-radius: 3px">
            Duplicating or Cloning a pilot will remove any Vault links, saving the copied Pilot as a
            new character on your account
          </div>
        </v-col>
      </v-row>
      <v-row dense justify="space-around">
        <v-col cols="5" class="text-center">
          <cc-tooltip
            inline
            title="Duplication"
            content="This will create a copy of this pilot data, registered to your pilot roster."
          >
            <v-btn large block color="primary" @click="copyPilot">Duplicate Pilot</v-btn>
          </cc-tooltip>
        </v-col>
        <v-col cols="5" class="text-center">
          <cc-tooltip
            inline
            title="Clone"
            content="This will clone the selected pilot. Cloned characters can’t join a mission in progress, and cloned characters receive a random quirk. Additional cloning and subjectivity imprinting adds further quirks."
          >
            <v-btn large block color="primary" :disabled="!!quirk" @click="rollQuirk">
              Flash Clone Pilot
            </v-btn>
          </cc-tooltip>
        </v-col>
      </v-row>
      <v-scale-transition>
        <div v-if="quirk">
          <v-row align="center">
            <v-col>
              <div class="overline">CLONE QUIRK</div>
              <v-alert dense color="primary" outlined left>
                <div class="body-text stark--text font-weight-bold">{{ quirk }}</div>
              </v-alert>
            </v-col>
            <v-col cols="auto">
              <cc-tooltip simple content="Reroll quirk">
                <v-btn icon large @click="rollQuirk">
                  <v-icon color="secondary" large>mdi-dice-6</v-icon>
                </v-btn>
              </cc-tooltip>
            </v-col>
          </v-row>
          <v-row v-if="quirk" align="center" justify="center">
            <v-col cols="6">
              <v-btn large block tile color="secondary darken-2" @click="clonePilot">
                Decant Flash Clone
              </v-btn>
            </v-col>
          </v-row>
        </div>
      </v-scale-transition>
    </v-card-text>
  </cc-solo-dialog>
</template>

<script lang="ts">
import Vue from 'vue'
import { getModule } from 'vuex-module-decorators'
import { CompendiumStore } from '@/store'
import _ from 'lodash'
import { Pilot } from '@/class'

export default Vue.extend({
  name: 'clone-dialog',
  props: {
    pilot: {
      type: Object,
      required: true,
    },
  },
  data: () => ({
    quirk: null,
  }),
  methods: {
    show() {
      this.$refs.dialog.show()
    },
    hide() {
      this.quirk = null
      this.$refs.dialog.hide()
    },
    rollQuirk() {
      const compendium = getModule(CompendiumStore, this.$store)
      this.quirk = _.sample(compendium.Tables.quirks)
    },
    clonePilot() {
      const newPilot = Pilot.Deserialize(Pilot.Serialize(this.pilot))
      newPilot.RenewID()
      newPilot.IsLocallyOwned = true
      if (!this.pilot.Callsign.includes('※')) this.pilot.Callsign += '※'
      if (!this.pilot.Callsign.includes('※')) this.pilot.Name += '※'
      this.pilot.Heal()
      this.pilot.AddQuirk(this.quirk)
      for (const mech of newPilot.Mechs) {
        mech.RenewID()
      }
      this.$store.dispatch('addPilot', { pilot: newPilot, update: true })
      this.hide()
    },
    copyPilot() {
      const newPilot = Pilot.Deserialize(Pilot.Serialize(this.pilot))
      newPilot.RenewID()
      newPilot.IsLocallyOwned = true
      newPilot.Callsign += '″'
      newPilot.Name += ' (COPY)'
      newPilot.Status = 'ACTIVE'
      for (const mech of newPilot.Mechs) {
        mech.RenewID()
      }
      this.$store.dispatch('addPilot', { pilot: newPilot, update: true })
      this.hide()
    },
  },
})
</script>
