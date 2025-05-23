<template>
  <ItemsListing
    itemtype="podcasts"
    path="librarypodcasts"
    :show-duration="false"
    :show-provider="true"
    :show-favorites-only-filter="true"
    :load-paged-data="loadItems"
    :show-library="true"
    :sort-keys="sortKeys"
    :update-available="updateAvailable"
    :title="$t('podcasts')"
    :allow-key-hooks="true"
    :show-search-button="true"
    icon="mdi-podcast"
    :restore-state="true"
    :total="total"
    :extra-menu-items="[
      {
        label: 'sync_now',
        icon: 'mdi-sync',
        action: () => {
          api.startSync([MediaType.PODCAST]);
        },
        overflowAllowed: true,
        disabled: api.syncTasks.value.length > 0,
      },
    ]"
  />
</template>

<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from "vue";
import ItemsListing, { LoadDataParams } from "@/components/ItemsListing.vue";
import api from "@/plugins/api";
import { EventMessage, EventType, MediaType } from "@/plugins/api/interfaces";
import { sleep } from "@/helpers/utils";
import { store } from "@/plugins/store";

defineOptions({
  name: "Podcasts",
});

const updateAvailable = ref(false);
const total = ref(store.libraryPodcastsCount);

const sortKeys = [
  "name",
  "name_desc",
  "sort_name",
  "sort_name_desc",
  "timestamp_added",
  "timestamp_added_desc",
  "timestamp_modified",
  "timestamp_modified_desc",
  "last_played",
  "last_played_desc",
  "play_count",
  "play_count_desc",
];

const loadItems = async function (params: LoadDataParams) {
  updateAvailable.value = false;
  setTotals(params);
  return await api.getLibraryPodcasts(
    params.favoritesOnly || undefined,
    params.search,
    params.limit,
    params.offset,
    params.sortBy,
  );
};

const setTotals = async function (params: LoadDataParams) {
  if (!params.favoritesOnly) {
    total.value = store.libraryPodcastsCount;
    return;
  }
  total.value = await api.getLibraryPodcastsCount(
    params.favoritesOnly || false,
  );
};

onMounted(() => {
  // signal if/when items get added within this library
  const unsub = api.subscribe(
    EventType.MEDIA_ITEM_ADDED,
    (evt: EventMessage) => {
      // signal user that there might be updated info available for this item
      if (evt.object_id?.startsWith("library://podcast")) {
        updateAvailable.value = true;
      }
    },
  );
  onBeforeUnmount(unsub);
});
</script>
