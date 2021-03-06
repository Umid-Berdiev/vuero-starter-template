<script setup lang="ts">
import { ref, computed, onMounted, h } from 'vue'
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { useHead } from '@vueuse/head'

import type {
  VFlexTableWrapperSortFunction,
  VFlexTableWrapperFilterFunction,
} from '/@src/components/base/table/VFlexTableWrapper.vue'
import { users } from '/@src/stores/usersMockData'
import ActionButtons from '/@src/components/partials/buttons/ActionButtons.vue'
import { useViewWrapper } from '/@src/stores/viewWrapper'
import ApplicantFormModal from '/@src/components/base/modal/ApplicantFormModal.vue'
import CountrySelect from '/@src/components/forms/selects/CountrySelect.vue'

const { t } = useI18n()

useHead({
  title: t('Applicants') + ' - Nefrit',
})

const viewWrapper = useViewWrapper()
viewWrapper.setPageTitle(t('Applicants_List'))

type User = typeof users[0]

// duplicate user data to grow the array
const data: User[] = users
// for (let i = 0; i < 100; i++) {
//   data.push(...users)
// }
const filterForm = ref({})

const isFormModalOpen = ref(false)
const displayFilterForm = ref(false)
const selectedRowsId = ref<number[]>([])
const isAllSelected = computed(() => data.length === selectedRowsId.value.length)
const router = useRouter()

// this is a sample for custom sort function
const locationSorter: VFlexTableWrapperSortFunction<User> = ({ order, a, b }) => {
  if (order === 'asc') {
    return a.location.localeCompare(b.location)
  } else if (order === 'desc') {
    return b.location.localeCompare(a.location)
  }

  return 0
}

// this is a sample for custom filter function
const userFilter: VFlexTableWrapperFilterFunction<User> = ({ searchTerm, row }) => {
  if (!searchTerm) {
    return true
  }

  // search either in the name or the bio
  return (
    row.name.toLocaleLowerCase().includes(searchTerm.toLocaleLowerCase()) ||
    row.bio.toLocaleLowerCase().includes(searchTerm.toLocaleLowerCase())
  )
}

const columns = {
  select: {
    label: '',
    cellClass: 'is-flex-grow-0',
  },
  orderNumber: {
    // label: '#',
    format: (value, row, index) => `${index + 1}`,
    cellClass: 'is-flex-grow-0',
  },
  name: {
    label: t('Applicant_user_name'),
    // media: true,
    // grow: true,
    searchable: true,
    sortable: true,
    filter: userFilter,
  },
  shortname: {
    label: t('Boss_name'),
    // media: true,
    // grow: true,
    searchable: true,
    sortable: true,
    filter: userFilter,
  },
  company: {
    label: t('Company_name'),
    // media: true,
    // grow: true,
    searchable: true,
    sortable: true,
    filter: userFilter,
  },
  phone: {
    label: t('Phone_number'),
    sortable: true,
    searchable: true,
    sort: locationSorter,
  },
  status: {
    label: t('Status'),
    sortable: true,
    searchable: true,
    sort: locationSorter,
  },
  location: t('Location'),
  actions: {
    label: t('Actions'),
    align: 'center',
  },
} as const

// the select all checkbox click handler
function toggleSelection() {
  // console.log('data:', data)

  if (isAllSelected.value) {
    selectedRowsId.value = []
  } else {
    selectedRowsId.value = data.map((row: any) => row.id)
  }
}

// this it the row click handler (enabled with clickable props)
function clickOnRow(row: any) {
  if (selectedRowsId.value.includes(row.id)) {
    selectedRowsId.value = selectedRowsId.value.filter((i) => i !== row.id)
  } else {
    selectedRowsId.value = [...selectedRowsId.value, row.id]
  }
}

function onActionTriggered(rowId) {
  router.push('/app/applicant/' + rowId)
}
</script>

<template>
  <div class="applicant-list-wrapper">
    <TableActionsBlock center title="" @add="isFormModalOpen = true" @filter="displayFilterForm = !displayFilterForm" />
    <div v-show="displayFilterForm" class="mb-5">
      <VCard radius="small">
        <h3 class="title is-5 mb-2">{{ t('Filter_form') }}</h3>
        <div class="columns is-desktop">
          <VField class="column">
            <VLabel>{{ t('Applicant_user_name') }}</VLabel>
            <VControl>
              <VInput v-model="filterForm.applicantUser" type="text" placeholder="" />
            </VControl>
          </VField>
          <VField class="column">
            <VLabel>{{ t('Status') }}</VLabel>
            <VControl>
              <VInput v-model="filterForm.applicantStatus" type="text" placeholder="" />
            </VControl>
          </VField>
          <VField class="column">
            <VLabel>{{ t('Boss_name') }}</VLabel>
            <VControl>
              <VInput v-model="filterForm.applicantBossName" type="text" placeholder="" />
            </VControl>
          </VField>
          <!-- </div>
              <div class="columns is-desktop"> -->
          <VField class="column">
            <VLabel>{{ t('Applicant_name') }}</VLabel>
            <VControl>
              <VInput v-model="filterForm.applicantName" type="text" placeholder="" />
            </VControl>
          </VField>
          <VField class="column">
            <VLabel>{{ t('Phone') }}</VLabel>
            <VControl>
              <VInput v-model="filterForm.applicantPhone" type="text" placeholder="" />
            </VControl>
          </VField>
          <CountrySelect v-model="filterForm.applicantsCountry" class="column" />
        </div>
        <div class="columns">
          <div class="column is-1 ml-auto">
            <VButton outlined color="warning" fullwidth icon="feather:filter">{{ t('Filter') }}</VButton>
          </div>
        </div>
      </VCard>
    </div>

    <!-- table -->
    <VFlexTableWrapper :columns="columns" :data="data">
      <!--
        Here we retrieve the internal wrapperState.
        Note that we can not destructure it
      -->
      <template #default="wrapperState">
        <!-- We can place any content inside the default slot-->
        <VFlexTableToolbar>
          <template #left>
            <!-- We can bind wrapperState.searchInput to any input -->
            <VField>
              <VControl icon="feather:search">
                <VInput v-model="wrapperState.searchInput" class="is-rounded" :placeholder="t('Search') + '...'" />
              </VControl>
            </VField>
          </template>
        </VFlexTableToolbar>

        <VFlexTable rounded>
          <!-- header-column slot -->
          <template #header-column="{ column }">
            <VCheckbox v-if="column.key === 'select'" class="ml-2 mr-3" :checked="isAllSelected" name="all_selected"
              color="primary" @click="toggleSelection" />
            <span v-if="column.key === 'orderNumber'" class="is-flex-grow-0" v-text="'#'" />
          </template>

          <!-- Custom "name" cell content -->
          <template #body-cell="{ row, column, value, index }">
            <VCheckbox v-if="column.key === 'select'" v-model="selectedRowsId" :value="row.id" name="selection"
              @change="clickOnRow" />

            <template v-else-if="column.key === 'status'">
              <VTag class="is-size-6" rounded :color="
                value === 'pending'
                  ? 'warning'
                  : value === 'rejected'
                    ? 'danger'
                    : value === 'completed'
                      ? 'primary'
                      : undefined
              ">
                {{ t(value) }}
              </VTag>
            </template>
            <template v-if="column.key === 'actions'">
              <ActionButtons @view="onActionTriggered(row.id)" @edit="isFormModalOpen = true" />
            </template>
          </template>
        </VFlexTable>

        <!-- Table Pagination with wrapperState.page binded-->
        <VFlexPagination v-model:current-page="wrapperState.page" class="mt-6" :item-per-page="wrapperState.limit"
          :total-items="wrapperState.total" :max-links-displayed="5" no-router />
      </template>
    </VFlexTableWrapper>
    <ApplicantFormModal v-model="isFormModalOpen" />
  </div>
</template>
