<template>
  <div class="total-cost">
    <div ref="costRefContainer" class="total-cost__value">
      <div class="total-cost__value-container" :class="getStyles" ref="costRefText">
        <span>{{ normalizedCost }}</span>
      </div>
    </div>
    <div v-if="normalizedDate" class="total-cost__date">
      <span>по состоянию на {{ normalizedDate }}</span>
    </div>
    <teleport to="body">
      <ul
          v-if="isPopupValid"
          :style="getPosition"
          class="total-cost__popup"
      >
        <li
            v-for="(currency, i) in normalizedCurrencies"
            :key="i"
            @click="handleSelect(currency.value)"
        >
          {{ currency.text }}
        </li>
      </ul>
    </teleport>
  </div>
</template>

<script>
export const ALLOWED_CURRENCY_LIST = [
  { value: 'RUB', text: 'В рублях' },
  { value: 'USD', text: 'В долларах США' },
  { value: 'EUR', text: 'В евро' },
]

export default {
  name: 'Dropdown',
  props: {
    items: {
      type: Array,
      default: () => [],
    },
    cost: {
      type: Number,
      default: 0,
    },
    date: {
      type: String,
      default: '',
    },
    item: {
      type: String,
      default: '',
    },
  },
  data () {
    return {
      isVisible: false,
      position: {
        bottomContainer: null,
        leftContainer: null,
      },
    }
  },
  emits: ['selectCurrency'],
  computed: {
    getPosition () {
      return {
        top: this.$data.position.bottomContainer + pageYOffset + 'px',
        left: this.$data.position.leftContainer + 'px',
      }
    },
    normalizedDate () {
      const date = new Date(this.$props.date)
      if (isNaN(date.getTime())) {
        return ''
      }
      return date.toLocaleDateString('ru-RU')
    },
    normalizedCost () {
      const formatter = new Intl.NumberFormat('ru-RU', {
        style: 'currency',
        minimumFractionDigits: 0,
        currency: this.$props.item,
      })
      return formatter.format(this.$props.cost)
    },
    normalizedCurrencies () {
      const currencies = []
      ALLOWED_CURRENCY_LIST.forEach((item) => {
        if (this.$props.items.includes(item.value)) {
          currencies.push(item)
        }
      })
      return currencies
    },
    isPopupValid () {
      return this.$data.isVisible && this.normalizedCurrencies.length > 1
    },
    getStyles () {
      return {
        visible: this.$props.items.length > 1,
        open: this.$data.isVisible && this.$props.items.length > 1,
      }
    },
  },
  methods: {
    showMenu () {
      this.$data.isVisible = true
    },
    hideMenu () {
      this.$data.isVisible = false
    },
    handleSelect (item) {
      this.$emit('selectCurrency', item)
    },
    handleClick (e) {
      const isCostClicked = this.$refs.costRefText.contains(e.target)
      if (!isCostClicked) {
        this.hideMenu()
      } else {
        if (this.isVisible) {
          this.hideMenu()
        } else {
          this.showMenu()
        }
      }
    },
    updatePosition () {
      const { bottom, left } = this.$refs.costRefContainer.getBoundingClientRect()

      this.$data.position.bottomContainer = bottom
      this.$data.position.leftContainer = left
    },
    handleResize () {
      this.updatePosition()
    },
  },
  beforeMount () {
    document.addEventListener('click', this.handleClick, { capture: true })
    window.addEventListener('resize', this.handleResize)
    window.addEventListener('scroll', this.handleResize)
  },
  updated () {
    this.updatePosition()
  },
  beforeUnmount () {
    document.removeEventListener('click', this.handleClick, { capture: true })
    window.removeEventListener('resize', this.handleResize)
    window.removeEventListener('scroll', this.handleResize)
  },
}
</script>

<style lang="scss" scoped>
.total-cost {
  &__value {
    letter-spacing: -0.02em;
  }

  &__date {
    letter-spacing: -0.005em;
    margin-top: 8px;
  }

  &__popup {
    position: absolute;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    width: 257px;
    border-radius: 12px;
    overflow: hidden;

    li {
      flex-basis: 100%;
      padding: 12px 16px;
      text-align: left;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      cursor: pointer;
      letter-spacing: -0.02em;
    }
  }
}

.total-cost__icon {
  display: flex;
  transition: transform 0.2s;
}

.total-cost__value-container {
  display: inline-flex;
  flex-direction: row;
  flex-wrap: nowrap;
  align-items: center;
  pointer-events: none;
  -webkit-tap-highlight-color: transparent;

  &.visible {
    pointer-events: unset;
    cursor: pointer;
  }

  &:not(.visible) .total-cost__icon {
    display: none;
  }

  &.visible.open .total-cost__icon {
    transform: rotate(180deg);
  }
}
</style>
