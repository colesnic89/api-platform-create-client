<template>
  <Toolbar
    :actions="['delete']"
    :breadcrumb="breadcrumb"
    :is-loading="isLoading"
    @delete="deleteItem"
  />

  <v-container fluid>
    <v-alert v-if="error || deleteError" type="error" class="mb-4" closable="true">
      \{{ error || deleteError }}
    </v-alert>

    <v-table v-if="item">
      <thead>
        <tr>
          <th>\{{ $t("field") }}</th>
          <th>\{{ $t("value") }}</th>
        </tr>
      </thead>

      <tbody>
        {{#each fields}}
        <tr>
          <td>
            \{{ $t("{{../lc}}.{{name}}") }}
          </td>

          <td>
            {{#if isReferences}}
            <template v-if="router.hasRoute('{{reference.title}}Show')">
              <router-link
                v-for="{{lowercase reference.title}} in item.{{reference.name}}"
                :to="{ name: '{{reference.title}}Show', params: { id: {{lowercase reference.title}} } }"
                :key="{{lowercase reference.title}}"
              >
                \{{ {{lowercase reference.title}} }}

                <br />
              </router-link>
            </template>

            <template v-else>
              <p
                v-for="{{lowercase reference.title}} in item.{{reference.name}}"
                :key="{{lowercase reference.title}}"
              >
                \{{ {{lowercase reference.title}} }}
              </p>
            </template>
            {{else if reference}}
            <router-link
              v-if="router.hasRoute('{{reference.title}}Show')"
              :to="{ name: '{{reference.title}}Show', params: { id: item.{{lowercase reference.title}} } }"
            >
              \{{ item.{{lowercase reference.title}} }}
            </router-link>

            <p v-else>
              \{{ item.{{lowercase reference.title}} }}
            </p>
            {{else if isEmbeddeds}}
            <template v-if="router.hasRoute('{{embedded.title}}Show')">
              <router-link
                v-for="{{lowercase embedded.title}} in item.{{embedded.name}}"
                :to="{ name: '{{embedded.title}}Show', params: { id: {{lowercase embedded.title}}['@id'] } }"
                :key="{{lowercase embedded.title}}['@id']"
              >
                \{{ {{lowercase embedded.title}}["@id"] }}

                <br />
              </router-link>
            </template>

            <template v-else>
              <p
                v-for="{{lowercase embedded.title}} in item.{{embedded.name}}"
                :key="{{lowercase embedded.title}}['@id']"
              >
                \{{ {{lowercase embedded.title}}["@id"] }}
              </p>
            </template>
            {{else if embedded}}
            <router-link
              v-if="router.hasRoute('{{embedded.title}}Show')"
              :to="{ name: '{{embedded.title}}Show', params: { id: item.{{lowercase embedded.title}}?.['@id'] } }"
            >
              \{{ item.{{lowercase embedded.title}}?.["@id"] }}
            </router-link>

            <p v-else>
              \{{ item.{{lowercase embedded.title}}?.["@id"] }}
            </p>
            {{else if (compare type "==" "dateTime") }}
            \{{ formatDateTime(item.{{name}}) }}
            {{else}}
            \{{ item.{{name}} }}
            {{/if}}
          </td>
        </tr>
        {{/each}}
      </tbody>
    </v-table>
  </v-container>

  <Loading :visible="isLoading" />
</template>

<script setup lang="ts">
import { onBeforeUnmount } from "vue";
import { useI18n } from "vue-i18n";
import { useRoute, useRouter } from "vue-router";
import { storeToRefs } from "pinia";
import Toolbar from "@/components/common/Toolbar.vue";
import Loading from "@/components/common/Loading.vue";
import { useMercureItem } from "@/composables/mercureItem";
import { use{{titleUcFirst}}DeleteStore } from "@/stores/{{lc}}/delete";
import { use{{titleUcFirst}}ShowStore } from "@/stores/{{lc}}/show";
{{#if hasDateField}}
import { formatDateTime } from "@/utils/date";
{{/if}}
import { useBreadcrumb } from "@/composables/breadcrumb";

const { t } = useI18n();
const route = useRoute();
const router = useRouter();
const breadcrumb = useBreadcrumb();

const {{lc}}ShowStore = use{{titleUcFirst}}ShowStore();
const { retrieved: item, isLoading, error } = storeToRefs({{lc}}ShowStore);

const {{lc}}DeleteStore = use{{titleUcFirst}}DeleteStore();
const { deleted, error: deleteError } = storeToRefs({{lc}}DeleteStore);

useMercureItem({
  store: {{lc}}ShowStore,
  deleteStore: {{lc}}DeleteStore,
  redirectRouteName: "{{titleUcFirst}}List",
});

await {{lc}}ShowStore.retrieve(decodeURIComponent(route.params.id as string));

async function deleteItem() {
  if (!item?.value) {
    {{lc}}DeleteStore.setError(t("itemNotFound"));
    return;
  }

  await {{lc}}DeleteStore.deleteItem(item.value);

  if (!deleted?.value) {
    return;
  }

  router.push({ name: "{{titleUcFirst}}List" });
}

onBeforeUnmount(() => {
  {{lc}}ShowStore.$reset();
});
</script>
