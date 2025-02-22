<template>
  <div>
    <!-- OVERLAY -->
    <sw-transition type="fade">
      <div
        v-show="isSidebarOpen"
        class="fixed top-0 left-0 z-20 w-full h-full"
        style="background: rgba(48, 75, 88, 0.5)"
        @click.prevent="toggleSidebar"
      ></div>
    </sw-transition>

    <!-- DESKTOP MENU -->
    <div
      class="hidden w-56 h-screen pb-32 overflow-y-auto bg-white border-r border-gray-200 border-solid xl:w-64 sw-scroll md:block"
    >
      <sw-list
        v-for="(menuItems, groupIndex) in menuItems"
        :key="groupIndex"
        variant="sidebar"
      >
        <div v-for="(item, index) in menuItems" :key="index">
          <div v-if="item.route">
            <sw-list-item
              :title="$t(item.title)"
              :active="hasActiveUrl(item.route)"
              :to="item.route"
              tag-name="router-link"
            >
              <component slot="icon" :is="item.icon" class="h-5" />
            </sw-list-item>
          </div>
          <div v-else class="sidebar-link-list-group">
            <span class="sidebar-link-list-group-icon">
              <component slot="icon" :is="item.icon" class="h-5" />
            </span>
            <sw-list-group :title="$t(item.title)">
              <sw-list-item
                v-for="(subItem, subItemIndex) in item.routes"
                :title="$t(subItem.title)"
                :key="subItemIndex"
                :active="hasActiveUrl(subItem.route)"
                :to="subItem.route"
                class="sidebar-link-items"
                tag-name="router-link"
              >
              </sw-list-item>
            </sw-list-group>
          </div>
        </div>
      </sw-list>
    </div>

    <!-- MOBILE MENU -->
    <transition
      enter-class="-translate-x-full"
      enter-active-class="transition duration-300 ease-in-out transform"
      enter-to-class="translate-x-0"
      leave-active-class="transition duration-300 ease-in-out transform"
      leave-class="translate-x-0"
      leave-to-class="-translate-x-full"
    >
      <div
        v-show="isSidebarOpen"
        class="fixed top-0 z-30 w-64 h-screen pt-16 pb-32 overflow-y-auto bg-white border-r border-gray-200 border-solid sw-scroll md:hidden"
      >
        <sw-list
          v-for="(menuItems, groupIndex) in menuItems"
          :key="groupIndex"
          variant="sidebar"
        >
          <div v-for="(item, index) in menuItems" :key="index">
            <div v-if="item.route">
              <sw-list-item
                :title="$t(item.title)"
                :active="hasActiveUrl(item.route)"
                :to="item.route"
                tag-name="router-link"
              >
                <component slot="icon" :is="item.icon" class="h-5" />
              </sw-list-item>
            </div>
            <div v-else class="sidebar-link-list-group">
              <span class="sidebar-link-list-group-icon">
                <component slot="icon" :is="item.icon" class="h-5" />
              </span>
              <sw-list-group :title="$t(item.title)">
                <sw-list-item
                  v-for="(subItem, subItemIndex) in item.routes"
                  :title="$t(subItem.title)"
                  :key="subItemIndex"
                  :active="hasActiveUrl(subItem.route)"
                  :to="subItem.route"
                  class="sidebar-link-items"
                  tag-name="router-link"
                >
                </sw-list-item>
              </sw-list-group>
            </div>
          </div>
        </sw-list>
      </div>
    </transition>
  </div>
</template>

<script type="text/babel">
import {
  HomeIcon,
  UserIcon,
  StarIcon,
  DocumentIcon,
  DocumentTextIcon,
  CreditCardIcon,
  CalculatorIcon,
  ChartBarIcon,
  CogIcon,
  UsersIcon,
  OfficeBuildingIcon,
} from '@vue-hero-icons/outline'
import { mapGetters, mapActions } from 'vuex'

export default {
  components: {
    HomeIcon,
    UserIcon,
    StarIcon,
    DocumentIcon,
    DocumentTextIcon,
    CreditCardIcon,
    CalculatorIcon,
    ChartBarIcon,
    CogIcon,
    UsersIcon,
    OfficeBuildingIcon,
  },

  computed: {
    ...mapGetters(['isSidebarOpen']),
    ...mapGetters('user', ['currentUser']),
    menuItems() {
      let menu = [
        [
          {
            title: 'navigation.dashboard',
            icon: 'home-icon',
            route: '/admin/dashboard',
          },
          {
            title: 'navigation.contacts',
            icon: 'user-icon',
            routes: [
              {
                title: 'navigation.contacts_customers',
                route: '/admin/contacts/customers',
              },
              {
                title: 'navigation.contacts_suppliers',
                route: '/admin/contacts/suppliers',
              },
            ],
          },
          {
            title: 'navigation.items',
            icon: 'star-icon',
            route: '/admin/items',
          },
        ],
        [
          {
            title: 'navigation.estimates',
            icon: 'document-icon',
            route: '/admin/estimates',
          },
          {
            title: 'navigation.sales',
            icon: 'document-text-icon',
            route: '/admin/sales',
          },
          {
            title: 'navigation.payments',
            icon: 'credit-card-icon',
            route: '/admin/payments',
          },
          {
            title: 'navigation.expenses',
            icon: 'calculator-icon',
            route: '/admin/expenses',
          },
        ],
        [
          {
            title: 'navigation.reports',
            icon: 'chart-bar-icon',
            route: '/admin/reports',
          },
          {
            title: 'navigation.settings',
            icon: 'cog-icon',
            route: '/admin/settings',
          },
        ],
      ]

      if (this.currentUser.role == 'super admin') {
        menu[2] = [
          {
            title: 'navigation.companies',
            icon: 'office-building-icon',
            route: '/admin/companies',
          },
          ...menu[2],
        ]
      }

      // if (this.currentUser.role != 'super admin') {
      //   menu[0] = [
      //     ...menu[0],
      //     {
      //       title: 'navigation.contacts',
      //       icon: 'user-icon',
      //       routes: [
      //         {
      //           title: 'navigation.contacts_customers',
      //           route: '/admin/contacts/customers',
      //         },
      //         {
      //           title: 'navigation.contacts_suppliers',
      //           route: '/admin/contacts/suppliers',
      //         },
      //       ],
      //     },
      //   ]
      // }

      return menu
    },
  },

  methods: {
    ...mapActions(['toggleSidebar']),

    hasActiveUrl(url) {
      this.isActive = true
      return this.$route.path.indexOf(url) > -1
    },
    hasStaticUrl(url) {
      return this.$route.path.indexOf(url)
    },
  },
}
</script>
