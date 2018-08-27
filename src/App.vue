<template lang="pug">
  #app
    form#ingredients-form
      div
        label Ingredient 1
        select(name="ingredient-1-id", v-model="ingredient1Id")
          optgroup(label="Normal")
            option(v-for="ingredient in normalIngredients", :value="ingredient.id") {{ ingredient| format }}
          optgroup(label="Rare")
            option(v-for="ingredient in rareIngredients", :value="ingredient.id") {{ ingredient| format }}
          optgroup(label="Epic")
            option(v-for="ingredient in epicIngredients", :value="ingredient.id") {{ ingredient | format }}
      div
        label Ingredient 2
        select(name="ingredient-2-id", v-model="ingredient2Id")
          optgroup(label="Normal")
            option(v-for="ingredient in normalIngredients", :value="ingredient.id") {{ ingredient| format }}
          optgroup(label="Rare")
            option(v-for="ingredient in rareIngredients", :value="ingredient.id") {{ ingredient| format }}
          optgroup(label="Epic")
            option(v-for="ingredient in epicIngredients", :value="ingredient.id") {{ ingredient | format }}
      div
        label current state
        p Click to "cycle through" the "covered", "empty", "ingredient 1", "ingredient 2".
        #states
          div.grid
            div.row(v-for="(row, y) in currentState")
              div.cell(v-for="(url, x) in row")
                img(:src="cellToImg(url)", @click="changeStateAt(x, y)")
        button(type="button", @click="resetState") Reset state
      div
        button(type="button") Compute

    div#results
      p.error(v-if="constraintedCombinations.length === 0") {{ isDataInvalid || 'no possible combination found' }}
      template(v-for="c in constraintedCombinations")
        div.grid
          div.row(v-for="(row, y) in c")
            div.cell(v-for="(cell, x) in row")
              img(:src="cellToImg(cell)")
</template>

<script>
const resourceRoot = './';

function merge(grid1, grid2, ingredient1, ingredient2) {
  let ret = [
    new Array(5),
    new Array(5),
    new Array(5),
    new Array(5),
    new Array(5),
  ];

  for (let y = 0; y < 5 ; y++) {
    for (let x = 0; x < 5 ; x++) {
      let c1 = grid1[y][x], c2 = grid2[y][x];
      if (c1 === '_' && c2 === '_') {
        ret[y][x] = '_';
      } else if (c1 !== '_' && c2 === '_') {
        ret[y][x] = ingredient1.id;
      } else if (c2 !== '_' && c1 === '_') {
        ret[y][x] = ingredient2.id;
      } else {
        return null;
      }
    }
  }

  return ret;
}

function canFulfill(grid, constraints) {
  for (let i = 0 ; i < constraints.length ; i++){
    let c = constraints[i];
    let cell = grid[c.y][c.x];
    if (cell !== c.expected) {
      return false;
    }
  }
  return true;
}

export default {
  name: 'app',
  data() {
    return {
      ingredient1Id: null,
      ingredient2Id: null,
      currentState: [
        {0: '', 1: '', 2: '', 3: '', 4: ''},
        {0: '', 1: '', 2: '', 3: '', 4: ''},
        {0: '', 1: '', 2: '', 3: '', 4: ''},
        {0: '', 1: '', 2: '', 3: '', 4: ''},
        {0: '', 1: '', 2: '', 3: '', 4: ''},
      ],
    }
  },
  filters: {
    format(ingredient) {
      return `${ingredient.name}, ${ingredient.level}, ${ingredient.icon}`;
    },
  },
  methods: {
    //updateStateImages() {
    //  for (let y = 0; y < 5; y++) {
    //    for (let x = 0; x < 5; x++) {
    //      this.currentStateImages[y][x] = this.cellToImg(this.currentState[y][x]);
    //    }
    //  }
    //},
    changeStateAt(x, y) {
      console.log('changeStateAt');
      let cell = this.currentState[y][x];
      switch(cell) {
        case '?': this.currentState[y][x] = '_'; break;
        case '_': this.currentState[y][x] = this.ingredient1.id; break;
        case this.ingredient1.id: this.currentState[y][x] = this.ingredient2.id; break;
        case this.ingredient2.id: this.currentState[y][x] = '?'; break;
        default:
          throw Error('Dont know how to change state.');
      }
      console.table(this.currentState);
      //this.currentStateImages[y][x] = this.cellToImg(
      //  this.currentState[y][x]
      //);
    },
    cellToImg(cell) {
      return this.symbols[cell];
    },
    resetState() {
      for (let y = 0; y < 5; y++) {
        for (let x = 0; x < 5; x++) {
          this.currentState[y][x] = '?';
        }
      }
    },
  },
  computed: {
    normalIngredients() {
      return this.$store.getters['ingredients/normalIngredients'];
    },
    rareIngredients() {
      return this.$store.getters['ingredients/rareIngredients'];
    },
    epicIngredients() {
      return this.$store.getters['ingredients/epicIngredients'];
    },
    ingredient1() {
      return this.$store.getters['ingredients/getById'](this.ingredient1Id);
    },
    ingredient2() {
      return this.$store.getters['ingredients/getById'](this.ingredient2Id);
    },
    isDataInvalid() {
      if (this.ingredient1Id === this.ingredient2Id) {
        return 'Two ingredients are the same.';
      }
      return null;
    },
    symbols() {
      let ret = {
        '?': `${resourceRoot}covered.png`,
        '_': `${resourceRoot}empty.png`,
      };
      if (this.ingredient1){
        ret[this.ingredient1.id] = `${resourceRoot}${this.ingredient1.image}`;
      }
      if (this.ingredient2) {
        ret[this.ingredient2.id] = `${resourceRoot}${this.ingredient2.image}`;
      }
      return ret;
    },

    combinations() {
      let combinations = [];
      if (this.isDataInvalid) {
        return combinations;
      }
      this.ingredient1.possibilities.forEach((p1) => {
        this.ingredient2.possibilities.forEach((p2) => {
          let mergedResult = merge(p1, p2, this.ingredient1, this.ingredient2);
          if (mergedResult) {
            combinations.push(mergedResult);
          }
        });
      });
      return combinations;
    },
    constraints() {
      // convert current state to constraints
      let constraints = [];
      let grid = this.currentState;
      for (let y = 0; y < 5; y++) {
        for (let x = 0; x < 5; x++) {
          let expected = grid[y][x];
          if (expected !== '?') {
            constraints.push({ x, y, expected });
          }
        }
      }
      return constraints;
    },
    constraintedCombinations() {
      console.log(this.constraints);
      return this.combinations.filter((c) => {
        return canFulfill(c, this.constraints);
      });
    },

    currentStateImages() {
      let ret = [
        ['', '', '', '', ''],
        ['', '', '', '', ''],
        ['', '', '', '', ''],
        ['', '', '', '', ''],
        ['', '', '', '', ''],
      ];
      for (let y = 0; y < 5; y++) {
        for (let x = 0; x < 5; x++) {
          ret[y][x] = this.cellToImg(this.currentState[y][x]);
        }
      }
      console.table(ret);
      return ret;
    }
  },
  mounted() {
    this.ingredient1Id = this.normalIngredients[0].id;
    this.ingredient2Id = this.normalIngredients[1].id;
    this.resetState();
    //this.updateStateImages();
  },
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;

  #states {
  }

  #results {
  }

  .grid {
    display: inline-block;
    margin-right: 10px;

    .row {
      height: 40px;
      .cell {
        display: inline-block;
        img {
          width: 40px;
          height: 40px;
        }
      }
    }
  }
}
</style>
