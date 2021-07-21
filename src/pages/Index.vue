<template>
  <q-page class="row items-center justify-evenly">
    <div class="col-md-6 col-xs-11">
      <div class="row">
        <div class="col text-center">
          <h3>Hvilke land kan du reise til?</h3>
          <h6>Søk etter land for å se hvilke innreiseregler som gjelder.</h6>
        </div>
      </div>
      <div class="row">
        <div class="col">
          <div class="q-mx-auto q-px-sm search-box" @click="focusInput()">
            <span
              id="searchStringBlock"
              ref="searchStringRef"
              class="editable"
              contenteditable="true"
              @input="searchStringUpdated"
              >{{ searchString }}
            </span>
            <span id="block2" class="non-editable"
              >{{ suggestedComplete }}{{ suggestedComplete ? '?' : '' }}</span
            >
          </div>
        </div>
      </div>
      <div class="row q-pt-xl">
        <div class="col text-center">
          <h5 v-if="!allCountryRegions">
            {{ selectedCountry ? selectedCountry.description : '' }}
          </h5>
          <div v-else>
            <h5>
              {{ selectedCountry ? selectedCountry.name : '' }} er inndelt i
              flere regioner.
            </h5>

            <q-expansion-item
              expand-separator
              label="Trykk for å se regioner"
              header-class="expansion-header"
            >
              <div v-for="country of allCountryRegions" :key="country.id">
                <div class="row">
                  <div
                    class="col-2 text-right q-pr-sm"
                    :style="{ color: getColor(country.value) }"
                  >
                    ●
                  </div>
                  <div class="col text-left">
                    {{ country.region }} - {{ country.description }}
                  </div>
                </div>
              </div>
            </q-expansion-item>
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<style lang="scss" scoped>
span {
  outline: 0px solid transparent;
  font-size: 28px;
}

.non-editable {
  color: #9c9c9c;
}

.search-box {
  background-color: #ddc52a;
  max-width: 400px;
  height: 40px;
  cursor: text;
  border-radius: 5px;
  border: 2px solid #a69521;
}

.expansion-header {
  cursor: pointer;
}
</style>

<script lang="ts">
import axios from 'axios';
import { defineComponent, ref, onMounted } from 'vue';

interface ICountry {
  id: string;
  description: string;
  name: string;
  region: string;
  value: string;
}

interface IResponseFromFhi {
  config: unknown;
  data: IData;
}

interface IData {
  data: IDataNested[];
}

interface IDataNested {
  data: ICountry[];
}

export default defineComponent({
  name: 'PageIndex',
  setup() {
    const selectedCountry = ref<ICountry>();
    const countries = ref<ICountry[]>([]);
    const countriesFiltered = ref<ICountry[]>([]);
    const searchString = ref('');
    const suggestedComplete = ref('');
    const searchStringRef = ref<HTMLSpanElement>();
    const allCountryRegions = ref<ICountry[]>();
    const viewRegions = ref(false);

    const getCountries = async () => {
      return axios
        .get('https://www.fhi.no/api/chartdata/excel/series/104110/latest')
        .then((response: IResponseFromFhi) => {
          countries.value = response.data.data[0].data;
        })
        .catch((err) => {
          console.log(err);
        });
    };

    onMounted(() => {
      searchStringRef.value?.addEventListener('keypress', (event) => {
        if (event.code === 'Enter') {
          event.preventDefault();

          const countryName =
            searchString.value.toLowerCase() +
            suggestedComplete.value.toLowerCase();
          const country = countries.value.find(
            (countryIterated) =>
              countryIterated.name.toLowerCase() === countryName
          );

          if (!country) {
            return;
          }

          selectedCountry.value = country;

          if (country?.region) {
            allCountryRegions.value = countries.value.filter(
              (countryIterated) => countryIterated.name === country.name
            );
          } else {
            allCountryRegions.value = undefined;
          }

          searchStringRef.value?.blur();
          if (searchStringRef.value?.innerHTML) {
            searchStringRef.value.innerHTML = country.name;
            suggestedComplete.value = '';
          }
        }
      });

      setInterval(() => {
        if (!searchString.value || searchString.value === '') {
          setRandomCountry();
        }
      }, 4000);
    });

    const setRandomCountry = () => {
      const randomIndex = Math.random() * countries.value.length;
      suggestedComplete.value = countries.value[Math.trunc(randomIndex)].name;
    };

    const setModel = (value: string) => {
      console.log('🚀 ~ file: Index.vue ~ line 72 ~ setModel ~ value', value);
      selectedCountry.value = countries.value.find(
        (country) => country.name === value
      );
    };

    const filterFn = (val: string, update: (n: () => void) => void) => {
      update(() => {
        const needle = val.toLocaleLowerCase();
        countriesFiltered.value = countries.value.filter(
          (v) => v.name.toLocaleLowerCase().indexOf(needle) > -1
        );
      });
    };

    getCountries().catch((error) => {
      console.log(error);
    });

    const searchStringUpdated = (event: InputEvent) => {
      console.log(
        '🚀 ~ file: Index.vue ~ line 204 ~ searchStringUpdated ~ event',
        event
      );
      if (event.inputType === 'insertParagraph') {
        return;
      }
      const stringElement = searchStringRef.value;
      searchString.value =
        stringElement?.innerHTML.replace('&nbsp;', ' ') ?? '';
      const search = stringElement?.innerHTML.toLowerCase();

      if (!search) {
        suggestedComplete.value = '';
        return;
      }

      const foundCountry = countries.value.find((country) =>
        country.name.toLowerCase().startsWith(search)
      );

      if (!foundCountry) {
        suggestedComplete.value = '';
        return;
      }

      suggestedComplete.value = foundCountry.name
        .toLowerCase()
        .split(search)[1];
    };

    const focusInput = () => {
      searchStringRef.value?.focus();
    };

    const toggleViewRegions = () => {
      viewRegions.value = !viewRegions.value;
    };

    const getColor = (colorCode: string) => {
      switch (colorCode) {
        case '2':
          return '#ffff00';
        case '5':
          return '#00ff00';
        case '6':
          return '#FFA500';
        case '7':
          return '#D3D3D3';
        case '8':
          return '#800080';
        default:
          return '#000000';
      }
    };

    return {
      selectedCountry,
      countriesFiltered,
      setModel,
      filterFn,
      searchString,
      suggestedComplete,
      searchStringUpdated,
      searchStringRef,
      focusInput,
      allCountryRegions,
      viewRegions,
      toggleViewRegions,
      getColor,
    };
  },
});
</script>
