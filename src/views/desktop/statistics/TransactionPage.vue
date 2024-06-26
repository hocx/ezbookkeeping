<template>
    <v-row class="match-height">
        <v-col cols="12">
            <v-card>
                <v-layout>
                    <v-navigation-drawer :permanent="alwaysShowNav" v-model="showNav">
                        <div class="mx-6 my-4">
                            <btn-vertical-group :disabled="loading" :buttons="[
                                { name: $t('Categorical Analysis'), value: 'categoricalAnalysis' },
                            ]" v-model="activeTab" />
                        </div>
                        <v-divider />
                        <div class="mx-6 mt-4" v-if="activeTab === 'categoricalAnalysis'">
                            <span class="text-subtitle-2">{{ $t('Chart Type') }}</span>
                            <v-select
                                item-title="name"
                                item-value="type"
                                class="mt-2"
                                density="compact"
                                :disabled="loading"
                                :items="allChartTypesArray"
                                v-model="queryChartType"
                            >
                                <template #selection="{ item }">
                                    <span class="cursor-pointer">{{ $t(item.title) }}</span>
                                </template>

                                <template #item="{ props, item }">
                                    <v-list-item v-bind="props" :title="$t(item.title)"></v-list-item>
                                </template>
                            </v-select>
                        </div>
                        <div class="mx-6 mt-4" v-if="activeTab === 'categoricalAnalysis'">
                            <span class="text-subtitle-2">{{ $t('Sort Order') }}</span>
                            <v-select
                                item-title="name"
                                item-value="type"
                                class="mt-2"
                                density="compact"
                                :disabled="loading"
                                :items="allSortingTypesArray"
                                v-model="querySortingType"
                            >
                                <template #selection="{ item }">
                                    <span class="cursor-pointer">{{ $t(item.title) }}</span>
                                </template>

                                <template #item="{ props, item }">
                                    <v-list-item v-bind="props" :title="$t(item.title)"></v-list-item>
                                </template>
                            </v-select>
                        </div>
                        <v-tabs show-arrows class="my-4" direction="vertical"
                                :disabled="loading" v-model="query.chartDataType"
                                v-if="activeTab === 'categoricalAnalysis'">
                            <v-tab class="tab-text-truncate" :key="dataType.type" :value="dataType.type"
                                   v-for="dataType in allChartDataTypes">
                                <span class="text-truncate">{{ $t(dataType.name) }}</span>
                                <v-tooltip activator="parent" location="right">{{ $t(dataType.name) }}</v-tooltip>
                            </v-tab>
                        </v-tabs>
                    </v-navigation-drawer>
                    <v-main>
                        <v-window class="d-flex flex-grow-1 disable-tab-transition w-100-window-container" v-model="activeTab">
                            <v-window-item value="categoricalAnalysis">
                                <v-card variant="flat" min-height="680">
                                    <template #title>
                                        <div class="title-and-toolbar d-flex align-center">
                                            <v-btn class="mr-3 d-md-none" density="compact" color="default" variant="plain"
                                                   :ripple="false" :icon="true" @click="showNav = !showNav">
                                                <v-icon :icon="icons.menu" size="24" />
                                            </v-btn>
                                            <span>{{ $t('Statistics & Analysis') }}</span>
                                            <v-btn-group class="ml-4" color="default" density="comfortable" variant="outlined" divided>
                                                <v-btn :icon="icons.left"
                                                       :disabled="loading || query.dateType === allDateRanges.All.type || query.chartDataType === allChartDataTypes.AccountTotalAssets.type || query.chartDataType === allChartDataTypes.AccountTotalLiabilities.type"
                                                       @click="shiftDateRange(query.startTime, query.endTime, -1)"/>
                                                <v-menu location="bottom">
                                                    <template #activator="{ props }">
                                                        <v-btn :disabled="loading || query.chartDataType === allChartDataTypes.AccountTotalAssets.type || query.chartDataType === allChartDataTypes.AccountTotalLiabilities.type"
                                                               v-bind="props">{{ dateRangeName(query) }}</v-btn>
                                                    </template>
                                                    <v-list>
                                                        <v-list-item :key="dateRange.type" :value="dateRange.type"
                                                                     :append-icon="(query.dateType === dateRange.type ? icons.check : null)"
                                                                     v-for="dateRange in allDateRanges">
                                                            <v-list-item-title
                                                                class="cursor-pointer"
                                                                @click="setDateFilter(dateRange.type)">
                                                                {{ $t(dateRange.name) }}
                                                                <div class="statistics-custom-datetime-range" v-if="dateRange.type === allDateRanges.Custom.type && query.dateType === allDateRanges.Custom.type && query.startTime && query.endTime">
                                                                    <span>{{ queryStartTime }}</span>
                                                                    <span>&nbsp;-&nbsp;</span>
                                                                    <br/>
                                                                    <span>{{ queryEndTime }}</span>
                                                                </div>
                                                            </v-list-item-title>
                                                        </v-list-item>
                                                    </v-list>
                                                </v-menu>
                                                <v-btn :icon="icons.right"
                                                       :disabled="loading || query.dateType === allDateRanges.All.type || query.chartDataType === allChartDataTypes.AccountTotalAssets.type || query.chartDataType === allChartDataTypes.AccountTotalLiabilities.type"
                                                       @click="shiftDateRange(query.startTime, query.endTime, 1)"/>
                                            </v-btn-group>

                                            <v-btn density="compact" color="default" variant="text" size="24"
                                                   class="ml-2" :icon="true" :loading="loading" @click="reload">
                                                <template #loader>
                                                    <v-progress-circular indeterminate size="20"/>
                                                </template>
                                                <v-icon :icon="icons.refresh" size="24" />
                                                <v-tooltip activator="parent">{{ $t('Refresh') }}</v-tooltip>
                                            </v-btn>
                                            <v-spacer/>
                                            <v-btn density="comfortable" color="default" variant="text" class="ml-2"
                                                   :disabled="loading" :icon="true">
                                                <v-icon :icon="icons.more" />
                                                <v-menu activator="parent">
                                                    <v-list>
                                                        <v-list-item :prepend-icon="icons.filter"
                                                                     :title="$t('Filter Accounts')"
                                                                     @click="showFilterAccountDialog = true"></v-list-item>
                                                        <v-list-item :prepend-icon="icons.filter"
                                                                     :title="$t('Filter Transaction Categories')"
                                                                     @click="showFilterCategoryDialog = true"></v-list-item>
                                                        <v-divider class="my-2"/>
                                                        <v-list-item to="/app/settings?tab=statisticsSetting"
                                                                     :prepend-icon="icons.filterSettings"
                                                                     :title="$t('Settings')"></v-list-item>
                                                    </v-list>
                                                </v-menu>
                                            </v-btn>
                                        </div>
                                    </template>

                                    <v-card-text class="statistics-overview-title pt-0" :class="{ 'disabled': loading }"
                                                 v-if="initing || (statisticsData && statisticsData.items && statisticsData.items.length)">
                                        <span class="statistics-subtitle">{{ totalAmountName }}</span>
                                        <span class="statistics-overview-amount ml-3"
                                              :class="statisticsTextColor"
                                              v-if="!initing && statisticsData && statisticsData.items && statisticsData.items.length">
                                            {{ getDisplayAmount(statisticsData.totalAmount, defaultCurrency) }}
                                        </span>
                                        <v-skeleton-loader class="skeleton-no-margin ml-3 mb-2"
                                                           width="120px" type="text" :loading="true"
                                                           v-else-if="initing"></v-skeleton-loader>
                                    </v-card-text>

                                    <v-card-text class="statistics-overview-title pt-0"
                                                 v-else-if="!initing && (!statisticsData || !statisticsData.items || !statisticsData.items.length)">
                                        <span class="statistics-subtitle statistics-overview-empty-tip">{{ $t('No transaction data') }}</span>
                                    </v-card-text>

                                    <v-card-text :class="{ 'readonly': loading }" v-if="query.chartType === allChartTypes.Pie">
                                        <pie-chart
                                            :items="[
                                                {id: '1', name: '---', value: 60, color: '7c7c7f'},
                                                {id: '2', name: '---', value: 20, color: 'a5a5aa'},
                                                {id: '3', name: '---', value: 20, color: 'c5c5c9'}
                                            ]"
                                            :skeleton="true"
                                            id-field="id"
                                            name-field="name"
                                            value-field="value"
                                            color-field="color"
                                            v-if="initing"
                                        ></pie-chart>
                                        <pie-chart
                                            :items="statisticsData && statisticsData.items && statisticsData.items.length ? statisticsData.items : []"
                                            :min-valid-percent="0.0001"
                                            :show-value="showAmountInChart"
                                            :enable-click-item="true"
                                            :default-currency="defaultCurrency"
                                            id-field="id"
                                            name-field="name"
                                            value-field="totalAmount"
                                            percent-field="percent"
                                            currency-field="currency"
                                            hidden-field="hidden"
                                            v-else-if="!initing"
                                            @click="clickPieChartItem"
                                        />
                                    </v-card-text>

                                    <v-card-text :class="{ 'readonly': loading }" v-if="query.chartType === allChartTypes.Bar">
                                        <v-list rounded lines="two" v-if="initing">
                                            <template :key="itemIdx" v-for="itemIdx in [ 1, 2, 3 ]">
                                                <v-list-item class="pl-0">
                                                    <template #prepend>
                                                        <div>
                                                            <v-icon class="disabled mr-0" size="34" :icon="icons.square" />
                                                        </div>
                                                    </template>
                                                    <div class="d-flex flex-column ml-2">
                                                        <div class="d-flex">
                                                            <v-skeleton-loader class="skeleton-no-margin my-2"
                                                                               width="120px" type="text" :loading="true"></v-skeleton-loader>
                                                        </div>
                                                        <div>
                                                            <v-progress-linear :model-value="0" :height="4"></v-progress-linear>
                                                        </div>
                                                    </div>
                                                </v-list-item>
                                                <v-divider v-if="itemIdx < 3"/>
                                            </template>
                                        </v-list>
                                        <v-list class="py-0" rounded lines="two" v-else-if="!initing && statisticsData && statisticsData.items && statisticsData.items.length">
                                            <template :key="idx"
                                                      v-for="(item, idx) in statisticsData.items">
                                                <v-list-item class="pl-0" v-if="!item.hidden">
                                                    <template #prepend>
                                                        <router-link class="statistics-list-item" :to="getItemLinkUrl(item)">
                                                            <ItemIcon :icon-type="queryChartDataCategory" size="34px"
                                                                      :icon-id="item.icon"
                                                                      :color="item.color"></ItemIcon>
                                                        </router-link>
                                                    </template>
                                                    <router-link class="statistics-list-item" :to="getItemLinkUrl(item)">
                                                        <div class="d-flex flex-column ml-2">
                                                            <div class="d-flex">
                                                                <span>{{ item.name }}</span>
                                                                <small class="statistics-percent" v-if="item.percent >= 0">{{ getDisplayPercent(item.percent, 2, '&lt;0.01') }}</small>
                                                                <v-spacer/>
                                                                <span class="statistics-amount">{{ getDisplayAmount(item.totalAmount, (item.currency || defaultCurrency)) }}</span>
                                                            </div>
                                                            <div>
                                                                <v-progress-linear :color="item.color ? '#' + item.color : 'primary'"
                                                                                   :bg-color="isDarkMode ? '#161616' : '#f8f8f8'" :bg-opacity="1"
                                                                                   :model-value="item.percent >= 0 ? item.percent : 0"
                                                                                   :height="4"></v-progress-linear>
                                                            </div>
                                                        </div>
                                                    </router-link>
                                                </v-list-item>
                                                <v-divider v-if="!item.hidden && idx !== statisticsData.items.length - 1"/>
                                            </template>
                                        </v-list>
                                    </v-card-text>
                                </v-card>
                            </v-window-item>
                        </v-window>
                    </v-main>
                </v-layout>
            </v-card>
        </v-col>
    </v-row>

    <date-range-selection-dialog :title="$t('Custom Date Range')"
                                  :min-time="query.startTime"
                                  :max-time="query.endTime"
                                  v-model:show="showCustomDateRangeDialog"
                                  @dateRange:change="setCustomDateFilter" />

    <v-dialog width="800" v-model="showFilterAccountDialog">
        <account-filter-settings-card
            :dialog-mode="true" :modify-default="false"
            @settings:change="showFilterAccountDialog = false" />
    </v-dialog>

    <v-dialog width="800" v-model="showFilterCategoryDialog">
        <category-filter-settings-card
            :dialog-mode="true" :modify-default="false"
            @settings:change="showFilterCategoryDialog = false" />
    </v-dialog>

    <snack-bar ref="snackbar" />
</template>

<script>
import { useDisplay, useTheme } from 'vuetify';

import { mapStores } from 'pinia';
import { useSettingsStore } from '@/stores/setting.js';
import { useUserStore } from '@/stores/user.js';
import { useAccountsStore } from '@/stores/account.js';
import { useTransactionCategoriesStore } from '@/stores/transactionCategory.js';
import { useStatisticsStore } from '@/stores/statistics.js';

import datetimeConstants from '@/consts/datetime.js';
import statisticsConstants from '@/consts/statistics.js';
import { limitText, formatPercent } from '@/lib/common.js'
import {
    getShiftedDateRangeAndDateType,
    getDateRangeByDateType
} from '@/lib/datetime.js';

import {
    mdiCheck,
    mdiArrowLeft,
    mdiArrowRight,
    mdiSort,
    mdiRefresh,
    mdiSquareRounded,
    mdiMenu,
    mdiFilterOutline,
    mdiFilterCogOutline,
    mdiPencilOutline,
    mdiDotsVertical
} from '@mdi/js';

import AccountFilterSettingsCard from './settings/cards/AccountFilterSettingsCard.vue';
import CategoryFilterSettingsCard from './settings/cards/CategoryFilterSettingsCard.vue';

export default {
    components: {
        AccountFilterSettingsCard,
        CategoryFilterSettingsCard
    },
    data() {
        const { mdAndUp } = useDisplay();

        return {
            activeTab: 'categoricalAnalysis',
            initing: true,
            loading: true,
            alwaysShowNav: mdAndUp.value,
            showNav: mdAndUp.value,
            showCustomDateRangeDialog: false,
            showFilterAccountDialog: false,
            showFilterCategoryDialog: false,
            icons: {
                check: mdiCheck,
                left: mdiArrowLeft,
                right: mdiArrowRight,
                sort: mdiSort,
                refresh: mdiRefresh,
                square: mdiSquareRounded,
                menu: mdiMenu,
                filter: mdiFilterOutline,
                filterSettings: mdiFilterCogOutline,
                pencil: mdiPencilOutline,
                more: mdiDotsVertical
            }
        };
    },
    computed: {
        ...mapStores(useSettingsStore, useUserStore, useAccountsStore, useTransactionCategoriesStore, useStatisticsStore),
        isDarkMode() {
            return this.globalTheme.global.name.value === 'dark';
        },
        defaultCurrency() {
            return this.userStore.currentUserDefaultCurrency;
        },
        firstDayOfWeek() {
            return this.userStore.currentUserFirstDayOfWeek;
        },
        query() {
            return this.statisticsStore.transactionStatisticsFilter;
        },
        queryChartDataCategory() {
            return this.statisticsStore.transactionStatisticsChartDataCategory;
        },
        queryChartType: {
            get: function () {
                return this.query.chartType;
            },
            set: function(value) {
                this.setChartType(value);
            }
        },
        querySortingType: {
            get: function () {
                return this.query.sortingType;
            },
            set: function(value) {
                this.setSortingType(value);
            }
        },
        queryStartTime() {
            return this.$locale.formatUnixTimeToLongDateTime(this.userStore, this.query.startTime);
        },
        queryEndTime() {
            return this.$locale.formatUnixTimeToLongDateTime(this.userStore, this.query.endTime);
        },
        allChartTypes() {
            return statisticsConstants.allChartTypes;
        },
        allChartTypesArray() {
            return [
                {
                    name: 'Pie Chart',
                    type: this.allChartTypes.Pie
                },
                {
                    name: 'Bar Chart',
                    type: this.allChartTypes.Bar
                }
            ];
        },
        allChartDataTypes() {
            return statisticsConstants.allChartDataTypes;
        },
        allSortingTypes() {
            return statisticsConstants.allSortingTypes;
        },
        allSortingTypesArray() {
            return statisticsConstants.allSortingTypesArray;
        },
        allDateRanges() {
            return datetimeConstants.allDateRanges;
        },
        showAccountBalance() {
            return this.settingsStore.appSettings.showAccountBalance;
        },
        totalAmountName() {
            if (this.query.chartDataType === this.allChartDataTypes.IncomeByAccount.type
                || this.query.chartDataType === this.allChartDataTypes.IncomeByPrimaryCategory.type
                || this.query.chartDataType === this.allChartDataTypes.IncomeBySecondaryCategory.type) {
                return this.$t('Total Income');
            } else if (this.query.chartDataType === this.allChartDataTypes.ExpenseByAccount.type
                || this.query.chartDataType === this.allChartDataTypes.ExpenseByPrimaryCategory.type
                || this.query.chartDataType === this.allChartDataTypes.ExpenseBySecondaryCategory.type) {
                return this.$t('Total Expense');
            } else if (this.query.chartDataType === this.allChartDataTypes.AccountTotalAssets.type) {
                return this.$t('Total Assets');
            } else if (this.query.chartDataType === this.allChartDataTypes.AccountTotalLiabilities.type) {
                return this.$t('Total Liabilities');
            }

            return this.$t('Total Amount');
        },
        statisticsData() {
            return this.statisticsStore.statisticsData;
        },
        statisticsTextColor() {
            if (this.query.chartDataType === this.allChartDataTypes.ExpenseByAccount.type ||
                this.query.chartDataType === this.allChartDataTypes.ExpenseByPrimaryCategory.type ||
                this.query.chartDataType === this.allChartDataTypes.ExpenseBySecondaryCategory.type) {
                return 'text-expense';
            } else if (this.query.chartDataType === this.allChartDataTypes.IncomeByAccount.type ||
                this.query.chartDataType === this.allChartDataTypes.IncomeByPrimaryCategory.type ||
                this.query.chartDataType === this.allChartDataTypes.IncomeBySecondaryCategory.type) {
                return 'text-income';
            } else {
                return 'text-default';
            }
        },
        showAmountInChart() {
            if (!this.showAccountBalance
                && (this.query.chartDataType === this.allChartDataTypes.AccountTotalAssets.type || this.query.chartDataType === this.allChartDataTypes.AccountTotalLiabilities.type)) {
                return false;
            }

            return true;
        }
    },
    watch: {
        'query.chartDataType': function (newValue) {
            this.statisticsStore.updateTransactionStatisticsFilter({
                chartDataType: newValue
            });
        },
        'display.mdAndUp.value': function (newValue) {
            this.alwaysShowNav = newValue;

            if (!this.showNav) {
                this.showNav = newValue;
            }
        }
    },
    created() {
        const self = this;

        self.statisticsStore.initTransactionStatisticsFilter();

        Promise.all([
            self.accountsStore.loadAllAccounts({ force: false }),
            self.transactionCategoriesStore.loadAllCategories({ force: false })
        ]).then(() => {
            return self.statisticsStore.loadTransactionStatistics({
                force: false
            });
        }).then(() => {
            self.loading = false;
            self.initing = false;
        }).catch(error => {
            self.loading = false;
            self.initing = false;

            if (!error.processed) {
                self.$refs.snackbar.showError(error);
            }
        });
    },
    setup() {
        const display = useDisplay();
        const theme = useTheme();

        return {
            display: display,
            globalTheme: theme
        };
    },
    methods: {
        reload(force) {
            const self = this;
            let dispatchPromise = null;

            self.loading = true;

            if (self.query.chartDataType === self.allChartDataTypes.ExpenseByAccount.type ||
                self.query.chartDataType === self.allChartDataTypes.ExpenseByPrimaryCategory.type ||
                self.query.chartDataType === self.allChartDataTypes.ExpenseBySecondaryCategory.type ||
                self.query.chartDataType === self.allChartDataTypes.IncomeByAccount.type ||
                self.query.chartDataType === self.allChartDataTypes.IncomeByPrimaryCategory.type ||
                self.query.chartDataType === self.allChartDataTypes.IncomeBySecondaryCategory.type) {
                dispatchPromise = self.statisticsStore.loadTransactionStatistics({
                    force: force
                });
            } else if (self.query.chartDataType === self.allChartDataTypes.AccountTotalAssets.type ||
                self.query.chartDataType === self.allChartDataTypes.AccountTotalLiabilities.type) {
                dispatchPromise = self.accountsStore.loadAllAccounts({
                    force: force
                });
            }

            if (dispatchPromise) {
                dispatchPromise.then(() => {
                    self.loading = false;

                    if (force) {
                        self.$refs.snackbar.showMessage('Data has been updated');
                    }
                }).catch(error => {
                    self.loading = false;

                    if (!error.processed) {
                        self.$refs.snackbar.showError(error);
                    }
                });
            }
        },
        setChartType(chartType) {
            this.statisticsStore.updateTransactionStatisticsFilter({
                chartType: chartType
            });
        },
        setSortingType(sortingType) {
            if (sortingType < this.allSortingTypes.Amount.type || sortingType > this.allSortingTypes.Name.type) {
                return;
            }

            this.statisticsStore.updateTransactionStatisticsFilter({
                sortingType: sortingType
            });

            this.reload(null);
        },
        setDateFilter(dateType) {
            if (dateType === this.allDateRanges.Custom.type) { // Custom
                this.showCustomDateRangeDialog = true;
                return;
            } else if (this.query.dateType === dateType) {
                return;
            }

            const dateRange = getDateRangeByDateType(dateType, this.firstDayOfWeek);

            if (!dateRange) {
                return;
            }

            this.statisticsStore.updateTransactionStatisticsFilter({
                dateType: dateRange.dateType,
                startTime: dateRange.minTime,
                endTime: dateRange.maxTime
            });

            this.reload(null);
        },
        setCustomDateFilter(startTime, endTime) {
            if (!startTime || !endTime) {
                return;
            }

            this.statisticsStore.updateTransactionStatisticsFilter({
                dateType: this.allDateRanges.Custom.type,
                startTime: startTime,
                endTime: endTime
            });

            this.showCustomDateRangeDialog = false;

            this.reload(null);
        },
        shiftDateRange(startTime, endTime, scale) {
            if (this.query.dateType === this.allDateRanges.All.type) {
                return;
            }

            const newDateRange = getShiftedDateRangeAndDateType(startTime, endTime, scale, this.firstDayOfWeek);

            this.statisticsStore.updateTransactionStatisticsFilter({
                dateType: newDateRange.dateType,
                startTime: newDateRange.minTime,
                endTime: newDateRange.maxTime
            });

            this.reload(null);
        },
        dateRangeName(query) {
            if (query.chartDataType === this.allChartDataTypes.AccountTotalAssets.type ||
                query.chartDataType === this.allChartDataTypes.AccountTotalLiabilities.type) {
                return this.$t(this.allDateRanges.All.name);
            }

            return this.$locale.getDateRangeDisplayName(this.userStore, query.dateType, query.startTime, query.endTime);
        },
        clickPieChartItem(item) {
            this.$router.push(this.getItemLinkUrl(item));
        },
        getDisplayAmount(amount, currency, textLimit) {
            amount = this.getDisplayCurrency(amount, currency);

            if (!this.showAccountBalance
                && (this.query.chartDataType === this.allChartDataTypes.AccountTotalAssets.type
                    || this.query.chartDataType === this.allChartDataTypes.AccountTotalLiabilities.type)
            ) {
                return '***';
            }

            if (textLimit) {
                return limitText(amount, textLimit);
            }

            return amount;
        },
        getDisplayCurrency(value, currencyCode) {
            return this.$locale.getDisplayCurrency(value, currencyCode, {
                currencyDisplayMode: this.settingsStore.appSettings.currencyDisplayMode,
                enableThousandsSeparator: this.settingsStore.appSettings.thousandsSeparator,
                enableDecimalPoint: this.settingsStore.appSettings.decimalPoint,
            });
        },
        getDisplayPercent(value, precision, lowPrecisionValue) {
            return formatPercent(value, precision, lowPrecisionValue);
        },
        getItemLinkUrl(item) {
            return `/transaction/list?${this.statisticsStore.getTransactionListPageParams(item)}`;
        }
    }
}
</script>

<style>
.statistics-custom-datetime-range {
    font-size: 0.7rem;
    line-height: 1rem;
    color: rgba(var(--v-theme-on-background), var(--v-medium-emphasis-opacity)) !important;
}

.statistics-overview-title {
    line-height: 2rem !important;
    height: 46px;
    display: flex;
    align-items: flex-end;
}

.statistics-overview-amount {
    font-size: 1.5rem;
    overflow: hidden;
    text-overflow: ellipsis;
}

.statistics-subtitle {
    font-size: 1rem;
    line-height: 1.75rem
}

.statistics-overview-empty-tip {
    color: rgba(var(--v-theme-on-background), var(--v-medium-emphasis-opacity)) !important;
}

.statistics-list-item {
    color: var(--v-theme-on-default);
    font-size: 1rem !important;
    line-height: 1.75rem;
    overflow: hidden;
    text-overflow: ellipsis;
}

.statistics-list-item .statistics-percent {
    font-size: 0.75rem;
    opacity: 0.7;
    margin-left: 6px;
}

.statistics-list-item .statistics-amount {
    opacity: 0.8;
}
</style>
