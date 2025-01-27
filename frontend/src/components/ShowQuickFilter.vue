<template>
  <NestedPopover>
    <template #target>
      <Button :label="buttonLabel" :class="buttonClass">
        <template v-if="hideLabel">
          <FileSpreadsheetIcon class="h-4" />
        </template>
        <template v-if="!hideLabel" #prefix>
          <FileSpreadsheetIcon class="h-4" />
        </template>
      </Button>
    </template>
    <template #body="{ close }">
      <div
        class="my-2 rounded-lg border border-gray-100 bg-white p-1.5 shadow-xl"
      >
        <div>
          <Button
            class="w-full !justify-start"
            variant="ghost"
            @click="filter('new', 'New Leads Today')"
            :label="__('New Leads Today')"
          >
          </Button>
          <Button
            class="w-full !justify-start"
            variant="ghost"
            @click="filter('today', 'Today Follow Ups')"
            :label="__('Today Follow Ups')"
          >
          </Button>

          <Button
            class="w-full !justify-start"
            variant="ghost"
            @click="filter('newToday', 'New & Today')"
            :label="__('New & Today')"
          >
          </Button>
          <!-- <Button
            class="w-full !justify-start"
            variant="ghost"
            @click="filter('today&beyond', 'Today & Beyond')"
            :label="__('Today & Beyond')"
          >
          </Button> -->
          <Button
            class="w-full !justify-start"
            variant="ghost"
            @click="filter('all', 'All')"
            :label="__('All')"
          >
          </Button>
          <Button
            v-if="!is_default"
            class="w-full !justify-start !text-gray-600"
            variant="ghost"
            @click="resetToDefault()"
            :label="__('Reset to Default')"
          >
            <template #prefix>
              <RefreshIcon class="h-4" />
            </template>
          </Button>
        </div>
      </div>
    </template>
  </NestedPopover>
</template>

<script setup>
import FileSpreadsheetIcon from '@/components/Icons/FileSpreadsheetIcon.vue'
import NestedPopover from '@/components/NestedPopover.vue'
import { computed, ref, onMounted, watch } from 'vue'
import RefreshIcon from '@/components/Icons/RefreshIcon.vue'

const props = defineProps({
  doctype: {
    type: String,
    required: true,
  },
  hideLabel: {
    type: Boolean,
    default: false,
  },
})
const emit = defineEmits(['update'])
const selectedFilter = ref('')
const selectedFilterLabel = ref('')

const list = defineModel()

function filter(val, label) {
  if (!val) return
  selectedFilter.value = val
  selectedFilterLabel.value = label
  emit('update', val)
}

const resetToDefault = () => {
  selectedFilter.value = ''
  selectedFilterLabel.value = ''
  emit('update', '')
}

const buttonLabel = computed(() => {
  return selectedFilter.value
    ? `Follow Up Filter : ${selectedFilterLabel.value}`
    : 'Follow Up Filter'
})

const buttonClass = computed(() => {
  return selectedFilter.value ? 'bg-black text-white' : ''
})

watch(
  () => list.value.params,
  (val) => {
    if (!val) return
    if (val.or_filters) {
      if (val.or_filters == {}) {
        resetToDefault()
        return
      }
      var or_filters = val.or_filters
      var prevFilter = ''
      var prevFilterLabel = ''
      if (
        (or_filters.status === 'New' || or_filters.status === 'new') &&
        or_filters.next_contact_date
      ) {
        if (or_filters.next_contact_date[0] === '>=') {
          prevFilter = 'today&beyond'
          prevFilterLabel = 'Today & Beyond'
        } else {
          prevFilter = 'newToday'
          prevFilterLabel = 'New & Today'
        }
      } else if (or_filters.status === 'New') {
        prevFilter = 'new'
        prevFilterLabel = 'New Leads Today'
      } else if (or_filters.next_contact_date) {
        prevFilter = 'today'
        prevFilterLabel = 'Today Follow Ups'
      } else {
        prevFilter = 'all'
        prevFilterLabel = 'All'
      }

      if (
        prevFilter !== selectedFilter.value &&
        prevFilterLabel !== selectedFilterLabel.value
      ) {
        filter(prevFilter, prevFilterLabel)
      }
    } else {
      resetToDefault()
    }

    // or_filters = val.params.or_filters
    // console.log('or_filters', or_filters)
  },
)
</script>

<style scoped>
.bg-black {
  background-color: black;
}

.text-white {
  color: white;
}
</style>
