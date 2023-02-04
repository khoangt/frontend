<template>
  <main-layout title="Subscriptions">
    <div class="subscriptions">
      <div class="subscriptions__current">
        Current subscription: <b>{{ user?.subscription?.name }}</b>
      </div>

      <a-spin :delay="350" :spinning="loading" tip="Loading...">
        <ul class="subscriptions__list">
        <li
          v-for="item in list"
          class="subscriptions__item subscription"
          :class="{
              'subscription_current': user?.subscription?.name === item.name
            }"
        >
          <h3 class="subscription__name">
            {{ item.name }}
            <template v-if="user?.subscription?.name === item.name">
              (current)
            </template>
          </h3>
          <div class="subscription__price">
            <span class="subscription__currency">$</span>
            <span class="subscription__value">{{ item.price }}</span>
          </div>
          <div class="subscription__term">/ month</div>

          <ul class="subscription__features features">
            <li class="features__item feature">
              Max profiles: <b>{{ item.maxProfiles }}</b>
            </li>
            <li class="features__item feature">
              <template v-if="item.maxProfiles > 25">
                Priority Support
              </template>
              <template v-else-if="item.maxProfiles > 5">
                Basic Support
              </template>
              <template v-else-if="item.maxProfiles < 3">
                No Support
              </template>
            </li>
          </ul>

          <a-popconfirm
            v-if="user?.subscription?.name === item.name"
            placement="topLeft"
            ok-text="Yes"
            cancel-text="No"
            @confirm="onCancel"
          >
            <template #title>
              Are you sure?
            </template>
            <a-button class="subscription__switch">Cancel</a-button>
          </a-popconfirm>
          <a-button class="subscription__switch" @click="onSwitch" v-else>
            Switch
          </a-button>
        </li>
      </ul>
      </a-spin>
    </div>
  </main-layout>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { storeToRefs } from 'pinia'

import MainLayout from '@/layouts/MainLayout.vue'

import { useUserStore } from '@/stores/userStore.js'

import { getSubscriptions } from '@/api/user.js'

const userStore = useUserStore()
const list = ref([])
const loading = ref(false)

const { fetchUser } = userStore
const { user } = storeToRefs(userStore)

const fetchSubscriptions = async () => {
  try {
    loading.value = true
    const { data } = await getSubscriptions()
    list.value = data.filter((item) => item.price > 0)
  } catch (e) {
    console.error(e)
    toast.error(e.response?.data?.message || e.message)
  } finally {
    loading.value = false
  }
}

const onSwitch = () => {}

const onCancel = () => {}

onMounted(() => {
  fetchUser()
  fetchSubscriptions()
})
</script>

<style lang="scss">
.dark {
  .subscriptions {
    .subscription {
      background: #000 !important;
      color: #fff;
      border: 1px solid darken(#fff, 80%);

      &_current,
      &:hover {
        box-shadow: 5px 5px 5px 0 rgba(255, 255, 255, 0.35) !important;
      }
    }
  }
}
</style>

<style lang="scss" scoped>
.subscriptions {
  &__current {
    font-size: 20px;
  }

  &__list {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 5%;

    margin-top: 40px;

    list-style: none;

    @media screen and (max-width: 768px) {
      grid-template-columns: repeat(2, 1fr);
      gap: 30px;
    }

    @media screen and (max-width: 550px) {
      grid-template-columns: repeat(1, 1fr);
      justify-content: center;
    }
  }

  .subscription {
    display: flex;
    flex-direction: column;
    align-items: center;

    padding: 20px;

    border-radius: 20px;

    background: #fff;
    border: 1px solid lighten(#000, 80%);

    transition: all 0.4s;

    &_current,
    &:hover {
      transform: translateY(-15px) scale(1.04);

      box-shadow: 5px 5px 5px 0 rgba(0, 0, 0, 0.08);

      border: 1px solid lighten(#000, 90%);
    }

    &__name {
      margin-bottom: 5px;

      font-size: 40px;
      font-weight: 600;
      line-height: 1;

      &_current {
        font-size: 41px;
        font-weight: 700;
      }
    }

    &__price,
    &__term {
      line-height: 1;
    }

    &__value {
      font-size: 27px;
    }

    &__currency {
      font-size: 25px;
    }

    &__term {
      margin-bottom: 30px;

      font-size: 20px;
    }

    &__switch {
      width: 100px;
      height: 32px;
    }

    .features {
      margin-bottom: 20px;

      list-style: none;

      font-size: 20px;
    }
  }
}
</style>