<script setup lang="ts">
import { computed, ref } from "vue";
import data from "../pl_pl.json";
import dataSimple from "../pl_pl_simple.json";
import VOtpInput from "vue3-otp-input";

const otpInput = ref(null);
const words = ref([]);

const included1 = computed(() => universalIncluded(0));
const included2 = computed(() => universalIncluded(1));
const included3 = computed(() => universalIncluded(2));
const included4 = computed(() => universalIncluded(3));
const included5 = computed(() => universalIncluded(4));

const wrongPlace1 = computed(() => universalWrongPlace(0));
const wrongPlace2 = computed(() => universalWrongPlace(1));
const wrongPlace3 = computed(() => universalWrongPlace(2));
const wrongPlace4 = computed(() => universalWrongPlace(3));
const wrongPlace5 = computed(() => universalWrongPlace(4));

const universalIncluded = (place: number) =>
  words.value
    .flat()
    .filter(
      (letter: { status: string }, index: number) =>
        index % 5 === place && letter.status === "active"
    )
    .map((letter: { letter: any }) => letter.letter)
    .join("")[0] || "";

const universalWrongPlace = (place: number) =>
  words.value
    .flat()
    .filter(
      (letter: { status: string }, index: number) =>
        index % 5 === place && letter.status === "wrongPlace"
    )
    .map((letter: { letter: any }) => letter.letter);

const excluded = computed(() =>
  words.value
    .flat()
    .filter((letter: { status: string }) => {
      return (
        letter.status === "inactive" &&
        !wrongPlace1.value.includes(letter.letter) &&
        !wrongPlace2.value.includes(letter.letter) &&
        !wrongPlace3.value.includes(letter.letter) &&
        !wrongPlace4.value.includes(letter.letter) &&
        !wrongPlace5.value.includes(letter.letter)
      );
    })
    .map((letter: { letter: any }) => letter.letter)
    .join("")
);

const getBestChars = (input: any[]) => {
  return input.reduce((total: { [x: string]: number }, word: string) => {
    word.split("").forEach((char: string | number) => {
      if (total[char]) {
        total[char] += 1;
      } else {
        total[char] = 1;
      }
    });
    return total;
  }, {});
};

const unique = (str: string | any[]) => {
  let result = "";
  for (var i = 0; i < str.length; i++) {
    if (result.indexOf(str[i]) < 0) {
      result += str[i];
    }
  }
  return result;
};

const getBestWords = (input: any[]) => {
  const chars = getBestChars(input);
  return input.map((word: any) => {
    return [
      word,
      unique(word)
        .split("")
        .reduce((total, char) => {
          return total + chars[char];
        }, 0),
    ];
  });
};

const getData = (input: string[]) => {
  let dataTmp = input;

  if (included1.value) {
    dataTmp = dataTmp.filter(
      (word: string) => word.indexOf(included1.value) === 0
    );
  }
  if (included2.value) {
    dataTmp = dataTmp.filter(
      (word: string) => word.indexOf(included2.value) === 1
    );
  }
  if (included3.value) {
    dataTmp = dataTmp.filter(
      (word: string) => word.indexOf(included3.value) === 2
    );
  }
  if (included4.value) {
    dataTmp = dataTmp.filter(
      (word: string) => word.indexOf(included4.value) === 3
    );
  }
  if (included5.value) {
    dataTmp = dataTmp.filter(
      (word: string) => word.indexOf(included5.value) === 4
    );
  }

  if (wrongPlace1.value.length) {
    dataTmp = dataTmp.filter(
      (word: string) =>
        wrongPlace1.value.filter(
          (item: string) =>
            word.indexOf(item) !== 0 && word.indexOf(item) !== -1
        ).length === wrongPlace1.value.length
    );
  }
  if (wrongPlace2.value.length) {
    dataTmp = dataTmp.filter(
      (word: string) =>
        wrongPlace2.value.filter(
          (item: string) =>
            word.indexOf(item) !== 1 && word.indexOf(item) !== -1
        ).length === wrongPlace2.value.length
    );
  }
  if (wrongPlace3.value.length) {
    dataTmp = dataTmp.filter(
      (word: string) =>
        wrongPlace3.value.filter(
          (item: string) =>
            word.indexOf(item) !== 2 && word.indexOf(item) !== -1
        ).length === wrongPlace3.value.length
    );
  }
  if (wrongPlace4.value.length) {
    dataTmp = dataTmp.filter(
      (word: string) =>
        wrongPlace4.value.filter(
          (item: string) =>
            word.indexOf(item) !== 3 && word.indexOf(item) !== -1
        ).length === wrongPlace4.value.length
    );
  }
  if (wrongPlace5.value.length) {
    dataTmp = dataTmp.filter(
      (word: string) =>
        wrongPlace5.value.filter(
          (item: string) =>
            word.indexOf(item) !== 4 && word.indexOf(item) !== -1
        ).length === wrongPlace5.value.length
    );
  }

  if (excluded.value.length > 0) {
    dataTmp = dataTmp.filter((word: string | string[]) => {
      const tmp1 = excluded.value
        .split("")
        .map((checkedChar: string[]) => {
          return word.includes(checkedChar[0]);
        })
        .filter((i: boolean) => i == true);
      return tmp1.length === 0;
    });
  }

  return dataTmp;
};

const list = computed(() => {
  return getBestWords(getData(data)).sort(
    (item1: number[], item2: number[]) => item2[1] - item1[1]
  );
});

const listSimple = computed(() => {
  return getBestWords(getData(dataSimple)).sort(
    (item1: number[], item2: number[]) => item2[1] - item1[1]
  );
});

const listLimit = (input: any[]) => {
  return input.splice(0, 100);
};

const getStatus = (letter: string, index: number) => {
  if (
    (index === 0 && included1.value === letter) ||
    (index === 1 && included2.value === letter) ||
    (index === 2 && included3.value === letter) ||
    (index === 3 && included4.value === letter) ||
    (index === 4 && included5.value === letter)
  ) {
    return "active";
  } else {
    return "inactive";
  }
};

const handleOnComplete = (value: string) => {
  words.value = [
    ...(words.value || []),
    value.split("").map((letter, index) => ({
      letter: letter,
      status: getStatus(letter, index),
    })),
  ];
  otpInput.value.clearInput();
};

const clickItem = (index: number, index1: any) => {
  words.value = words.value.map((word, indexA) => {
    if (index === indexA) {
      return word.map(
        (letter: { letter: any; status: string }, indexB: any) => {
          if (index1 === indexB) {
            return {
              letter: letter.letter,
              status:
                letter.status === "inactive"
                  ? "active"
                  : letter.status === "active"
                  ? "wrongPlace"
                  : "inactive",
            };
          } else {
            return letter;
          }
        }
      );
    } else {
      return word;
    }
  });
};
</script>

<template>
  <div class="greetings">
    <p>Literalnie solver © haktywista</p>
    <p>Wpisz słowo, klikając w przyciski oznaczasz czy litera występuje</p>
  </div>
  <div style="width: 500px; padding-right: 20px">
    <table>
      <tr v-for="(item, index) in words" :key="index">
        <th v-for="(item1, index1) in item" :key="index + index1">
          <button :class="item1.status" @click="clickItem(index, index1)">
            {{ item1.letter }}
          </button>
        </th>
      </tr>
    </table>
    <div style="margin-left: 1px">
      <v-otp-input
        ref="otpInput"
        input-classes="otp-input"
        input-type="letter-numeric"
        input-mode="text"
        separator=""
        :num-inputs="5"
        :should-auto-focus="true"
        :is-input-num="false"
        @on-complete="handleOnComplete"
        :conditionalClass="['one', 'two', 'three', 'four', 'five']"
      />
    </div>
  </div>

  <div style="display: flex">
    <div style="padding-left: 20px; width: 150px">
      <p>{{ "podstawowe: " + listSimple.length }}</p>
      <div
        v-for="(item, index) in listLimit(listSimple)"
        :key="`k1${item[0] + index}`"
      >
        <span>{{ item[0] }}</span>
      </div>
    </div>

    <div style="width: 150px">
      <p>{{ "wszystkie: " + list.length }}</p>
      <div
        v-for="(item, index) in listLimit(list)"
        :key="`k2${item[0] + index}`"
      >
        <span>{{ item[0] }}</span>
      </div>
    </div>
  </div>
</template>

<style>
html {
  background: black;
  color: white;
  font-family: monospace;
}
button {
  border: none;
  border-radius: 5px;
  width: 30px;
  height: 30px;
  padding: 0;
  margin: 0;
}
.otp-input {
  border: none;
  border-radius: 5px;
  width: 30px;
  height: 30px;
  padding: 0;
  margin: 2px;
  text-align: center;
}
.inactive {
  background-color: lightgrey;
}
.active {
  background-color: lightgreen;
}
.wrongPlace {
  background-color: violet;
}
</style>
