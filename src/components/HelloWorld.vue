<script setup lang="ts">
import { computed, ref } from "vue";
import data from "../pl_pl.json";
import dataSimple from "../pl_pl_simple.json";

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

const included = ref("");
const excluded = ref("");
const included1 = ref("");
const included2 = ref("");
const included3 = ref("");
const included4 = ref("");
const included5 = ref("");

const resetState = () => {
  included.value = "";
  excluded.value = "";
  included1.value = "";
  included2.value = "";
  included3.value = "";
  included4.value = "";
  included5.value = "";
};
const getData = (input: string[]) => {
  let dataTmp = input;

  if (included1.value) {
    dataTmp = dataTmp.filter(
      (word: string) => word.split("")[0] === included1.value
    );
  }
  if (included2.value) {
    dataTmp = dataTmp.filter(
      (word: string) => word.split("")[1] === included2.value
    );
  }
  if (included3.value) {
    dataTmp = dataTmp.filter(
      (word: string) => word.split("")[2] === included3.value
    );
  }
  if (included4.value) {
    dataTmp = dataTmp.filter(
      (word: string) => word.split("")[3] === included4.value
    );
  }
  if (included5.value) {
    dataTmp = dataTmp.filter(
      (word: string) => word.split("")[4] === included5.value
    );
  }

  if (included.value.length > 0) {
    dataTmp = dataTmp.filter((word: string | string[]) => {
      return (
        included.value
          .split("")
          .map((checkedChar) => {
            return word.includes(checkedChar[0]);
          })
          .filter((i) => i == false).length === 0
      );
    });
  }

  if (excluded.value.length > 0) {
    dataTmp = dataTmp.filter((word: string | string[]) => {
      const tmp1 = excluded.value
        .split("")
        .map((checkedChar) => {
          return word.includes(checkedChar[0]);
        })
        .filter((i) => i == true);
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
</script>

<template>
  <div class="greetings">
    Literalnie solver v0.0.1 do szybkiego poruszania się po polach tekstowych
    używaj `tab` © haktywista
  </div>
  <div style="width: 500px; padding-right: 20px">
    <table>
      <tr>
        <th>Zawiera litery</th>
        <th>Nie zawiera liter</th>
        <th>1 litera</th>
        <th>2 litera</th>
        <th>3 litera</th>
        <th>4 litera</th>
        <th>5 litera</th>
        <th></th>
      </tr>
      <tr>
        <th><input style="width: 120px" v-model="included" /></th>
        <th><input style="width: 120px" v-model="excluded" /></th>
        <th><input style="width: 50px" v-model="included1" /></th>
        <th><input style="width: 50px" v-model="included2" /></th>
        <th><input style="width: 50px" v-model="included3" /></th>
        <th><input style="width: 50px" v-model="included4" /></th>
        <th><input style="width: 50px" v-model="included5" /></th>
        <th><button @click="resetState">Reset</button></th>
      </tr>
    </table>
  </div>

  <div style="display: flex">
    <div style="padding-left: 20px; width: 250px">
      <p>{{ "lista słów - podstawowe: " + listSimple.length }}</p>
      <p>{{}}</p>
      <div v-for="item in listLimit(listSimple)" :key="`k1${item[0]}`">
        <span>{{ item[0] }}</span>
      </div>
    </div>

    <div style="width: 250px">
      <p>{{ "lista słów - wszystkie: " + list.length }}</p>
      <div v-for="item in listLimit(list)" :key="`k2${item[0]}`">
        <span>{{ item[0] }}</span>
      </div>
    </div>
  </div>
</template>
