<template>
  <div>
    <v-expansion-panels
      accordion
      multiple
      v-model="expanded"
    >
      <v-expansion-panel
        v-for="category in macros"
        :key="`category-${category.name}`"
      >
        <v-expansion-panel-header>
          <template v-slot:actions>
            <v-icon small class="mr-2">
              $expand
            </v-icon>
          </template>
          <div>
            {{ category.name }}
            <v-chip x-small>{{ category.macros.length }}</v-chip>
            <app-btn
              @click.prevent.stop="handleEditCategory(category.id)"
              icon
              text
              x-small
              color=""
              class="ml-2"
            >
              <v-icon x-small>$cog</v-icon>
            </app-btn>
          </div>
        </v-expansion-panel-header>

        <v-expansion-panel-content>
          <app-macro-btn
            v-for="macro in category.macros"
            :macro="macro"
            :key="`category-${macro.name}`"
            @click="sendGcode($event, `${waits.onMacro}${macro.name}`)"
            :loading="hasWait(`${waits.onMacro}${macro.name}`)"
            :elevation="2"
            class="me-2 mb-2 float-left">
            {{ macro.name }}
          </app-macro-btn>
        </v-expansion-panel-content>

      </v-expansion-panel>

      <v-expansion-panel
        v-if="uncategorizedMacros && uncategorizedMacros.macros.length > 0"
      >
        <v-expansion-panel-header>
          <template v-slot:actions>
            <v-icon small class="mr-2">
              $expand
            </v-icon>
          </template>
          <div>
            {{ $t('app.general.label.uncategorized') }}
            <v-chip x-small class="ml-2">{{ uncategorizedMacros.visible }}</v-chip>
            <app-btn
              @click.prevent.stop="handleEditCategory('0')"
              icon
              text
              x-small
              color=""
              class="ml-2"
            >
              <v-icon x-small>$cog</v-icon>
            </app-btn>
          </div>
        </v-expansion-panel-header>

        <v-expansion-panel-content>
          <template v-for="macro in uncategorizedMacros.macros">
            <app-macro-btn
              v-if="macro.visible"
              :macro="macro"
              :key="`category-${macro.name}`"
              @click="sendGcode($event, `${waits.onMacro}${macro.name}`)"
              :loading="hasWait(`${waits.onMacro}${macro.name}`)"
              :elevation="2"
              class="me-2 mb-2 float-left">
              {{ macro.name }}
            </app-macro-btn>
          </template>
        </v-expansion-panel-content>

      </v-expansion-panel>

    </v-expansion-panels>

    <!-- <pre>expanded: {{ expanded }}</pre> -->
  </div>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import StateMixin from '@/mixins/state'
import { Waits } from '@/globals'
import { Macro } from '@/store/macros/types'

@Component({})
export default class Macros extends Mixins(StateMixin) {
  waits = Waits

  get macros () {
    return this.$store.getters['macros/getVisibleMacros']
  }

  get uncategorizedMacros () {
    const macros = this.$store.getters['macros/getMacrosByCategory']()
    const count = macros.length
    const visible = macros.filter((macro: Macro) => macro.visible).length
    return {
      macros,
      count,
      visible
    }
  }

  get expanded () {
    let expanded: number[] = this.$store.state.macros.expanded
    // Remove any indexes that may no longer exist.
    expanded = expanded.filter(i => i <= this.macros.length)
    return expanded
  }

  set expanded (val: number[]) {
    this.$store.dispatch('macros/saveExpanded', val)
  }

  handleEditCategory (id: number) {
    this.$router.push(`/settings/macros/${id}`)
  }
}
</script>
