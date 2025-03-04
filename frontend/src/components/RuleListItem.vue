<script setup lang="ts">
import { CListGroupItem } from "@coreui/bootstrap-vue";

import { TRACKING_RULES } from "../api/constants";

import type { Rule } from "../api/types";
import DestinationTag from "./DestinationTag.vue";
const props = defineProps<{
  rule: Rule;
}>();

const tracking_rule = TRACKING_RULES.find(
  (o) => o.name === props.rule.tracking_rule.name
);
</script>

<template>
  <CListGroupItem
    class="d-flex justify-content-between align-items-center list-group-item-action"
    :class="{ 'text-muted bg-light': props.rule.disabled }"
    style="--bs-bg-opacity: 0.4"
    v-if="tracking_rule"
  >
    <div>
      <div class="fw-bold fs-4">
        <router-link
          :to="`/rules/${props.rule.id}`"
          class="text-decoration-none"
          >{{ props.rule.name }}</router-link
        >
      </div>
      <div>
        {{ tracking_rule.prefixlabel
        }}<template
          v-if="
            rule.tracking_rule.params && rule.tracking_rule.params.length > 1
          "
          >s:</template
        ><template
          v-else-if="
            rule.tracking_rule.params && rule.tracking_rule.params.length == 1
          "
          >:</template
        >
        <template
          v-if="
            rule.tracking_rule.name == 'artifacts-owned' ||
            rule.tracking_rule.name == 'artifacts-group-owned' ||
            rule.tracking_rule.name == 'users-followed'
          "
        >
          <template
            v-for="(name, index) in rule.tracking_rule.params"
            :key="index"
            ><span v-if="index != 0">,</span
            ><strong>&nbsp;{{ name }}</strong></template
          >
        </template>
        <template v-if="rule.tracking_rule.name == 'artifacts-followed'">
          <template
            v-for="(artifact, index) in rule.tracking_rule.params"
            :key="index"
            ><span v-if="index != 0">,</span>
            <strong>{{ artifact.type }}/{{ artifact.name }}</strong></template
          >
        </template>
      </div>
    </div>
    <div>
      <template v-for="(gr, index) in rule.generation_rules" :key="index">
        <DestinationTag
          v-for="destination in gr.destinations"
          :destination="destination"
          :key="`${destination.protocol}:${destination.address}`"
        />
      </template>
    </div>
    <div>Notifications in the past 7 days: {{ rule.generated_last_week }}</div>
  </CListGroupItem>
</template>
