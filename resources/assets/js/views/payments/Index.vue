<template>
  <base-page class="payments">
    <sw-page-header :title="$t('payments.title')">
      <sw-breadcrumb slot="breadcrumbs">
        <sw-breadcrumb-item :title="$t('general.home')" to="dashboard" />
        <sw-breadcrumb-item :title="$tc('payments.payment', 2)" to="#" active />
      </sw-breadcrumb>

      <template slot="actions">
        <sw-button
          v-show="totalPayments"
          variant="primary-outline"
          size="lg"
          @click="toggleFilter"
        >
          {{ $t('general.filter') }}
          <component :is="filterIcon" class="w-4 h-4 ml-2 -mr-1" />
        </sw-button>

        <sw-button
          tag-name="router-link"
          to="payments/create"
          variant="primary"
          size="lg"
          class="ml-4"
        >
          <plus-icon class="w-6 h-6 mr-1 -ml-2" />
          {{ $t('payments.add_payment') }}
        </sw-button>
      </template>
    </sw-page-header>

    <slide-y-up-transition>
      <sw-filter-wrapper v-show="showFilters" class="mt-3">
        <sw-input-group
          :label="$t('payments.customer')"
          color="black-light"
          class="flex-1 mt-2"
        >
          <base-customer-select
            ref="customerSelect"
            @select="onSelectCustomer"
            @deselect="clearCustomerSearch"
          />
        </sw-input-group>

        <sw-input-group
          :label="$t('payments.payment_number')"
          class="flex-1 mt-2 lg:ml-6"
        >
          <sw-input
            v-model="filters.payment_number"
            :placeholder="$t(payments.payment_number)"
            name="payment_number"
          />
        </sw-input-group>

        <sw-input-group
          :label="$t('payments.payment_mode')"
          class="flex-1 mt-2 lg:ml-6"
        >
          <sw-select
            v-model="filters.payment_mode"
            :options="paymentModes"
            :searchable="true"
            :show-labels="false"
            :placeholder="$t('payments.payment_mode')"
            label="name"
          />
        </sw-input-group>

        <label
          class="absolute text-sm leading-snug text-gray-900 cursor-pointer"
          style="top: 10px; right: 15px"
          @click="clearFilter"
          >{{ $t('general.clear_all') }}</label
        >
      </sw-filter-wrapper>
    </slide-y-up-transition>

    <sw-empty-table-placeholder
      v-if="showEmptyScreen"
      :title="$t('payments.no_payments')"
      :description="$t('payments.list_of_payments')"
    >
      <capsule-icon class="mt-5 mb-4" />

      <sw-button
        slot="actions"
        tag-name="router-link"
        to="/admin/payments/create"
        size="lg"
        variant="primary-outline"
      >
        <plus-icon class="w-6 h-6 mr-1 -ml-2" />
        {{ $t('payments.add_new_payment') }}
      </sw-button>
    </sw-empty-table-placeholder>

    <div v-show="!showEmptyScreen" class="relative table-container">
      <div
        class="relative flex items-center justify-between h-10 mt-5 list-none border-b-2 border-gray-200 border-solid"
      >
        <p class="text-sm">
          {{ $t('general.showing') }}: <b>{{ payments.length }}</b>
          {{ $t('general.of') }} <b>{{ totalPayments }}</b>
        </p>

        <sw-transition type="fade">
          <sw-dropdown v-if="selectedPayments.length">
            <span
              slot="activator"
              class="flex block text-sm font-medium cursor-pointer select-none text-primary-400"
            >
              {{ $t('general.actions') }}
              <chevron-down-icon class="h-5" />
            </span>

            <sw-dropdown-item @click="removeMultiplePayments">
              <trash-icon class="h-5 mr-3 text-gray-600" />
              {{ $t('general.delete') }}
            </sw-dropdown-item>
          </sw-dropdown>
        </sw-transition>
      </div>

      <div class="absolute z-10 items-center pl-4 mt-2 select-none md:mt-12">
        <sw-checkbox
          v-model="selectAllFieldStatus"
          variant="primary"
          size="sm"
          class="hidden md:inline"
          @change="selectAllPayments"
        />

        <sw-checkbox
          v-model="selectAllFieldStatus"
          :label="$t('general.select_all')"
          variant="primary"
          size="sm"
          class="md:hidden"
          @change="selectAllPayments"
        />
      </div>

      <sw-table-component
        ref="table"
        :data="fetchData"
        :show-filter="false"
        table-class="table"
      >
        <sw-table-column
          :sortable="false"
          :filterable="false"
          cell-class="no-click"
        >
          <div slot-scope="row" class="relative block">
            <sw-checkbox
              :id="row.id"
              v-model="selectField"
              :value="row.id"
              variant="primary"
              size="sm"
            />
          </div>
        </sw-table-column>

        <sw-table-column
          :sortable="true"
          :label="$t('payments.date')"
          sort-as="payment_date"
          show="formattedPaymentDate"
        />

        <sw-table-column
          :sortable="true"
          :label="$t('payments.payment_number')"
          show="payment_number"
        >
          <template slot-scope="row">
            <span>{{ $t('payments.payment_number') }}</span>
            <router-link
              :to="{ path: `payments/${row.id}/view` }"
              class="font-medium text-primary-500"
            >
              {{ row.payment_number }}
            </router-link>
          </template>
        </sw-table-column>

        <sw-table-column
          :sortable="true"
          :label="$t('payments.customer')"
          show="name"
        />

        <sw-table-column
          :sortable="true"
          :label="$t('payments.payment_mode')"
          show="payment_mode"
        >
          <template slot-scope="row">
            <span>{{ $t('payments.payment_mode') }}</span>
            <span>
              {{
                row.payment_mode
                  ? row.payment_mode
                  : $t('payments.not_selected')
              }}
            </span>
          </template>
        </sw-table-column>

        <sw-table-column
          :sortable="true"
          :label="$t('payments.invoice')"
          sort-as="invoice_id"
          show="invoice_number"
        >
          <template slot-scope="row">
            <span>{{ $t('invoices.invoice_number') }}</span>
            <span>
              {{
                row.invoice_number
                  ? row.invoice_number
                  : $t('payments.no_invoice')
              }}
            </span>
          </template>
        </sw-table-column>

        <sw-table-column :sortable="true" :label="$t('payments.amount')">
          <template slot-scope="row">
            <span>{{ $t('payments.amount') }}</span>
            <div v-html="$utils.formatMoney(row.amount, row.user.currency)" />
          </template>
        </sw-table-column>

        <sw-table-column
          :sortable="false"
          :filterable="false"
          cell-class="action-dropdown"
        >
          <template slot-scope="row">
            <span>{{ $t('payments.action') }}</span>
            <sw-dropdown>
              <dot-icon slot="activator" />

              <sw-dropdown-item
                :to="`payments/${row.id}/edit`"
                tag-name="router-link"
              >
                <pencil-icon class="h-5 mr-3 text-gray-600" />
                {{ $t('general.edit') }}
              </sw-dropdown-item>

              <sw-dropdown-item
                :to="`payments/${row.id}/view`"
                tag-name="router-link"
              >
                <eye-icon class="h-5 mr-3 text-gray-600" />
                {{ $t('general.view') }}
              </sw-dropdown-item>

              <sw-dropdown-item @click="removePayment(row.id)">
                <trash-icon class="h-5 mr-3 text-gray-600" />
                {{ $t('general.delete') }}
              </sw-dropdown-item>
            </sw-dropdown>
          </template>
        </sw-table-column>
      </sw-table-component>
    </div>
  </base-page>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import CapsuleIcon from '@/components/icon/CapsuleIcon'
import {
  PlusIcon,
  FilterIcon,
  XIcon,
  ChevronDownIcon,
  EyeIcon,
  PencilIcon,
  TrashIcon,
} from '@vue-hero-icons/solid'

export default {
  components: {
    CapsuleIcon,
    PlusIcon,
    FilterIcon,
    XIcon,
    ChevronDownIcon,
    EyeIcon,
    PencilIcon,
    TrashIcon,
  },

  data() {
    return {
      showFilters: false,
      sortedBy: 'created_at',
      isRequestOngoing: true,

      filters: {
        customer: '',
        payment_mode: '',
        payment_number: '',
      },
    }
  },

  computed: {
    showEmptyScreen() {
      return !this.totalPayments && !this.isRequestOngoing
    },

    filterIcon() {
      return this.showFilters ? 'x-icon' : 'filter-icon'
    },

    ...mapGetters('customer', ['customers']),

    ...mapGetters('payment', [
      'selectedPayments',
      'totalPayments',
      'payments',
      'selectAllField',
      'paymentModes',
    ]),

    selectField: {
      get: function () {
        return this.selectedPayments
      },
      set: function (val) {
        this.selectPayment(val)
      },
    },

    selectAllFieldStatus: {
      get: function () {
        return this.selectAllField
      },
      set: function (val) {
        this.setSelectAllState(val)
      },
    },
  },

  watch: {
    filters: {
      handler: 'setFilters',
      deep: true,
    },
  },

  mounted() {
    this.fetchPaymentModes({ limit: 'all' })
  },

  destroyed() {
    if (this.selectAllField) {
      this.selectAllPayments()
    }
  },

  methods: {
    ...mapActions('payment', [
      'fetchPayments',
      'selectAllPayments',
      'selectPayment',
      'deletePayment',
      'deleteMultiplePayments',
      'setSelectAllState',
      'fetchPaymentModes',
    ]),

    async fetchData({ page, filter, sort }) {
      let data = {
        customer_id: this.filters.customer ? this.filters.customer.id : '',
        payment_method_id:
          this.filters.payment_mode !== null
            ? this.filters.payment_mode.id
            : '',
        payment_number: this.filters.payment_number,
        orderByField: sort.fieldName || 'payment_date',
        orderBy: sort.order || 'desc',
        page,
      }

      this.isRequestOngoing = true
      let response = await this.fetchPayments(data)
      this.isRequestOngoing = false

      return {
        data: response.data.payments.data,
        pagination: {
          totalPages: response.data.payments.last_page,
          currentPage: page,
          count: response.data.payments.count,
        },
      }
    },

    refreshTable() {
      this.$refs.table.refresh()
    },

    setFilters() {
      this.refreshTable()
    },

    clearFilter() {
      if (this.filters.customer) {
        this.$refs.customerSelect.$refs.baseSelect.removeElement(
          this.filters.customer
        )
      }

      this.filters = {
        customer: '',
        payment_mode: '',
        payment_number: '',
      }
    },

    toggleFilter() {
      if (this.showFilters) {
        this.clearFilter()
      }

      this.showFilters = !this.showFilters
    },

    onSelectCustomer(customer) {
      this.filters.customer = customer
    },

    async removePayment(id) {
      swal({
        title: this.$t('general.are_you_sure'),
        text: this.$tc('payments.confirm_delete'),
        icon: '/assets/icon/trash-solid.svg',
        buttons: true,
        dangerMode: true,
      }).then(async (willDelete) => {
        if (willDelete) {
          let res = await this.deletePayment({ ids: [id] })

          if (res.data.success) {
            window.toastr['success'](this.$tc('payments.deleted_message', 1))
            this.$refs.table.refresh()
            return true
          }

          window.toastr['error'](res.data.message)
          return true
        }
      })
    },

    async removeMultiplePayments() {
      swal({
        title: this.$t('general.are_you_sure'),
        text: this.$tc('payments.confirm_delete', 2),
        icon: '/assets/icon/trash-solid.svg',
        buttons: true,
        dangerMode: true,
      }).then(async (willDelete) => {
        if (willDelete) {
          let request = await this.deleteMultiplePayments()
          if (request.data.success) {
            window.toastr['success'](this.$tc('payments.deleted_message', 2))
            this.$refs.table.refresh()
          } else if (request.data.error) {
            window.toastr['error'](request.data.message)
          }
        }
      })
    },

    async clearCustomerSearch(removedOption, id) {
      this.filters.customer = ''
      this.refreshTable()
    },

    showModel(selectedRow) {
      this.selectedRow = selectedRow
      this.$refs.Delete_modal.open()
    },

    async removeSelectedItems() {
      this.$refs.Delete_modal.close()
      await this.selectedRow.forEach((row) => {
        this.deletePayment(this.id)
      })
      this.$refs.table.refresh()
    },
  },
}
</script>
