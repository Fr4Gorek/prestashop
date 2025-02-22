<template>
  <SfTabs :open-tab="1">
    <SfTab :title="$t('My orders')">
      <div v-if="currentOrder">
        <SfButton class="sf-button--text all-orders" @click="backToAllOrders">{{$t('All Orders')}}</SfButton>
        <div class="highlighted highlighted--total">
          <SfProperty
            :name="$t('Order ID')"
            :value="orderGetters.getId(currentOrder)"
            class="sf-property--full-width property"
          />
          <SfProperty
            :name="$t('Date')"
            :value="orderGetters.getDate(currentOrder)"
            class="sf-property--full-width property"
          />
          <SfProperty
            :name="$t('Status')"
            :value="orderGetters.getStatus(currentOrder)"
            class="sf-property--full-width property"
          />
          <SfProperty
            :name="$t('Total')"
            :value="$n(orderGetters.getPrice(currentOrder), 'currency')"
            class="sf-property--full-width property"
          />
        </div>
        <order-items :order='currentOrder'></order-items>
      </div>
      <SfLoader :class="{ loading }" :loading="loading">
        <div v-if='currentOrder === null'>
          <p class="message">
            {{ $t('Details and status orders') }}
          </p>
          <div v-if="ordersList.length == 0" class="no-orders">
            <p class="no-orders__title">{{ $t('You currently have no orders') }}</p>
            <SfButton class="no-orders__button">{{ $t('Start shopping') }}</SfButton>
          </div>
          <SfTable v-else class="orders">
            <SfTableHeading>
              <SfTableHeader
                v-for="tableHeader in tableHeaders"
                :key="tableHeader"
              >{{ tableHeader }}</SfTableHeader>
              <SfTableHeader class="orders__element--right" />
            </SfTableHeading>
            <SfTableRow v-for="order in ordersList" :key="orderGetters.getId(order)">
              <SfTableData v-e2e="'order-number'">{{ orderGetters.getId(order) }}</SfTableData>
              <SfTableData>{{ orderGetters.getDate(order) }}</SfTableData>
              <SfTableData>{{ $n(orderGetters.getPrice(order), 'currency') }}</SfTableData>
              <SfTableData>
                <span :class="getStatusTextClass(order)">{{ orderGetters.getStatus(order) }}</span>
              </SfTableData>
              <SfTableData class="orders__view orders__element--right">
                <SfButton class="sf-button--text desktop-only" @click="currentOrder = order">
                  {{ $t('View details') }}
                </SfButton>
              </SfTableData>
            </SfTableRow>
          </SfTable>
          <p>{{ $t('Total orders') }} - {{ totalOrders }}</p>
        </div>
      </SfLoader>
    </SfTab>
    <SfTab title="Returns">
      <p class="message">
        This feature is not implemented yet!
      </p>
    </SfTab>
  </SfTabs>
</template>

<script>
import OrderItems from '../../components/OrderItems.vue';
import {
  SfTabs,
  SfTable,
  SfButton,
  SfProperty,
  SfLink,
  SfLoader
} from '@storefront-ui/vue';
import { computed, ref } from '@nuxtjs/composition-api';
import { useUserOrder, orderGetters } from '@vue-storefront/prestashop';
import { AgnosticOrderStatus } from '@vue-storefront/core';
import { onSSR } from '@vue-storefront/core';

export default {
  name: 'OrederHistory',
  components: {
    SfTabs,
    SfTable,
    SfButton,
    SfProperty,
    SfLink,
    SfLoader,
    OrderItems
  },
  setup() {
    const { orders, search, loading } = useUserOrder();
    const currentOrder = ref(null);
    const ordersList = computed(()=> orders.value ? orderGetters.getOrdersListFiltered(orders.value) : []);
    const totalOrders = computed(() => orders.value ? orderGetters.getOrdersTotal(orders.value) : 0);
    onSSR(async () => {
      await search({orderId: null});
    });

    const tableHeaders = [
      'Order ID',
      'Payment date',
      'Amount',
      'Status'
    ];

    const getStatusTextClass = (order) => {
      const status = orderGetters.getStatus(order);
      switch (status) {
        case AgnosticOrderStatus.Open:
          return 'text-warning';
        case AgnosticOrderStatus.Complete:
          return 'text-success';
        default:
          return '';
      }
    };
    const backToAllOrders = async () => {
      currentOrder.value = null;
      await search({orderId: null});
    };
    return {
      tableHeaders,
      ordersList,
      totalOrders,
      getStatusTextClass,
      orderGetters,
      currentOrder,
      loading,
      backToAllOrders
    };
  }
};
</script>

<style lang='scss' scoped>
.no-orders {
  &__title {
    margin: 0 0 var(--spacer-lg) 0;
    font: var(--font-weight--normal) var(--font-size--base) / 1.6 var(--font-family--primary);
  }
  &__button {
    --button-width: 100%;
    @include for-desktop {
      --button-width: 17,5rem;
    }
  }
}
.orders {
  @include for-desktop {
    &__element {
      &--right {
        --table-column-flex: 1;
        text-align: right;
      }
    }
  }
}
.all-orders {
  --button-padding: var(--spacer-base) 0;
}
.message {
  margin: 0 0 var(--spacer-xl) 0;
  font: var(--font-weight--light) var(--font-size--base) / 1.6 var(--font-family--primary);
  &__link {
    color: var(--c-primary);
    font-weight: var(--font-weight--medium);
    font-family: var(--font-family--primary);
    font-size: var(--font-size--base);
    text-decoration: none;
    &:hover {
      color: var(--c-text);
    }
  }
}
.product {
  &__properties {
    margin: var(--spacer-xl) 0 0 0;
  }
  &__property,
  &__action {
    font-size: var(--font-size--sm);
  }
  &__action {
    color: var(--c-gray-variant);
    font-size: var(--font-size--sm);
    margin: 0 0 var(--spacer-sm) 0;
    &:last-child {
      margin: 0;
    }
  }
  &__qty {
    color: var(--c-text);
  }
}
.products {
  --table-column-flex: 1;
  &__name {
    margin-right: var(--spacer-sm);
    @include for-desktop {
      --table-column-flex: 2;
    }
  }
}
.highlighted {
  box-sizing: border-box;
  width: 100%;
  background-color: var(--c-light);
  padding: var(--spacer-sm);
  --property-value-font-size: var(--font-size--base);
  --property-name-font-size: var(--font-size--base);
  &:last-child {
    margin-bottom: 0;
  }
  ::v-deep .sf-property__name {
    white-space: nowrap;
  }
  ::v-deep .sf-property__value {
    text-align: right;
  }
  &--total {
    margin-bottom: var(--spacer-sm);
  }
  @include for-desktop {
    padding: var(--spacer-xl);
    --property-name-font-size: var(--font-size--lg);
    --property-name-font-weight: var(--font-weight--medium);
    --property-value-font-size: var(--font-size--lg);
    --property-value-font-weight: var(--font-weight--semibold);
  }
}

</style>
