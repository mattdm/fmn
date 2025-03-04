<script setup lang="ts">
import { validationErrorToFormErrors } from "@/api";
import { useAddRuleMutation } from "@/api/rules";
import type { GenerationRule, PostError, Rule } from "@/api/types";
import { useToastStore } from "@/stores/toast";
import { CCol, CRow } from "@coreui/bootstrap-vue";
import type { FormKitNode } from "@formkit/core";
import { FormKit } from "@formkit/vue";
import type { AxiosError } from "axios";
import { computed, ref } from "vue";
import { useRouter } from "vue-router";
import GenerationRuleList from "./rule-edit/generation-rule/GenerationRuleList.vue";
import TrackingRule from "./rule-edit/tracking-rule/TrackingRule.vue";

const toastStore = useToastStore();
const router = useRouter();

const { mutateAsync } = useAddRuleMutation();

const trackingRuleName = ref("");
const value = ref("New Rule 1");

const generationRulesCount = ref(0);
const formReady = computed(
  () => trackingRuleName.value !== "" && generationRulesCount.value > 0
);

const handleSubmit = async (data: Rule, form: FormKitNode | undefined) => {
  console.log("Will submit the new rule:", data);
  if (!form) {
    throw Error("No form node?");
  }
  try {
    const response = await mutateAsync(data);
    // Success!
    toastStore.addToast({
      color: "success",
      title: "Rule created",
      content: `Rule "${response.name}" has been successfully created.`,
    });
    router.push("/");
  } catch (err) {
    const error = err as AxiosError<PostError>;
    console.log("Got error response from server:", error);
    if (!error.response) {
      return;
    }
    form.setErrors(validationErrorToFormErrors(error.response.data));
  }
};

const handleTrackingRuleSelected = (name: string) => {
  trackingRuleName.value = name;
};
const handleGenerationRulesChanged = (rules: GenerationRule[]) => {
  generationRulesCount.value = rules.length;
};
</script>

<template>
  <FormKit type="form" id="new-rule" @submit="handleSubmit" :actions="false">
    <CRow class="mb-2">
      <CCol xs="auto" class="flex-fill">
        <h4>
          <FormKit
            type="text"
            name="name"
            v-model="value"
            input-class="form-control-lg"
          />
        </h4>
      </CCol>
      <CCol xs="auto">
        <FormKit
          type="submit"
          :class="['btn', 'btn-primary', 'form-control-lg']"
          :disabled="!formReady"
        >
          Create Rule
        </FormKit>
      </CCol>
    </CRow>

    <CRow>
      <CCol sm="4" class="border-end">
        <TrackingRule @selected="handleTrackingRuleSelected" />
      </CCol>
      <CCol>
        <GenerationRuleList
          v-if="!!trackingRuleName"
          @change="handleGenerationRulesChanged"
        />
      </CCol>
    </CRow>
  </FormKit>
</template>
