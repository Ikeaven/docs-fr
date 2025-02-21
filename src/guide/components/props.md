# Props {#props}

> Cette page suppose que vous avez déjà lu les [principes fondamentaux des composants](/guide/essentials/component-basics). Lisez-les d'abord si vous débutez avec les composants.

<div class="options-api">
  <VueSchoolLink href="https://vueschool.io/lessons/vue-3-reusable-components-with-props" title="Free Vue.js Props Lesson"/>
</div>

## Déclaration de props {#props-declaration}

Les composants Vue nécessitent une déclaration explicite des props afin que Vue sache quels props externes passés au composant doivent être traités comme des attributs implicitement déclarés (qui seront discutés dans [sa section dédiée](/guide/components/attrs)).

<div class="composition-api">

Dans les SFC utilisant `<script setup>`, les props peuvent être déclarés à l'aide de la macro `defineProps()` :

```vue
<script setup>
const props = defineProps(['foo'])

console.log(props.foo)
</script>
```

Dans les composants autres que `<script setup>`, les props sont déclarés à l'aide de l'option [`props`](/api/options-state#props) :

```js
export default {
  props: ['foo'],
  setup(props) {
    // setup() reçoit des props comme premier argument.
    console.log(props.foo)
  }
}
```

Notez que l'argument passé à `defineProps()` est le même que la valeur fournie à l'option `props` : la même API de déclaration de props est partagée entre les deux styles de déclaration.

</div>

<div class="options-api">

Les props sont déclarés à l'aide de l'option [`props`](/api/options-state#props) :

```js
export default {
  props: ['foo'],
  created() {
    // les props sont exposés sur `this`
    console.log(this.foo)
  }
}
```

</div>

En plus de déclarer des props à l'aide d'un tableau de chaînes de caractères, nous pouvons également utiliser la syntaxe utilisant un objet :

<div class="options-api">

```js
export default {
  props: {
    title: String,
    likes: Number
  }
}
```

</div>
<div class="composition-api">

```js
// dans <script setup>
defineProps({
  title: String,
  likes: Number
})
```

```js
// dans un fichier autre que <script setup>
export default {
  props: {
    title: String,
    likes: Number
  }
}
```

</div>

Pour chaque propriété dans l'objet servant de déclaration, la clé est le nom du prop, tandis que la valeur doit être la fonction constructeur du type attendu.

Cela documente non seulement votre composant, mais avertira également, dans la console du navigateur, les autres développeurs utilisant votre composant s'ils transmettent le mauvais type. Nous discuterons plus en détail de la [validation de prop](#prop-validation) plus loin sur cette page.

<div class="options-api">

Voir aussi : [Typage de props de composant](/guide/typescript/options-api#typing-component-props) <sup class="vt-badge ts" />

</div>

<div class="composition-api">

Si vous utilisez TypeScript avec `<script setup>`, il est également possible de déclarer des props en utilisant uniquement des annotations de type :

```vue
<script setup lang="ts">
defineProps<{
  title?: string
  likes?: number
}>()
</script>
```

Plus de détails: [Typage de props de composant](/guide/typescript/composition-api#typing-component-props) <sup class="vt-badge ts" />

</div>

## Détails sur le passage de props {#prop-passing-details}

### Casse du nom de prop {#prop-name-casing}

Nous déclarons les noms de props en utilisant le camelCase car cela évite d'avoir à utiliser des guillemets lors de leur utilisation comme clés de propriété, et nous permet de les référencer directement dans les expressions de template car ce sont des identifiants JavaScript valides :

<div class="composition-api">

```js
defineProps({
  greetingMessage: String
})
```

</div>
<div class="options-api">

```js
export default {
  props: {
    greetingMessage: String
  }
}
```

</div>

```vue-html
<span>{{ greetingMessage }}</span>
```

Techniquement, vous pouvez également utiliser le camelCase lors de la transmission de props à un composant enfant (sauf dans les [templates DOM](/guide/essentials/component-basics#dom-template-parsing-caveats)). Cependant, la convention utilise le kebab-case dans tous les cas pour s'aligner sur les attributs HTML :

```vue-html
<MyComponent greeting-message="hello" />
```

Nous utilisons le [PascalCase pour les balises de composant](/guide/components/registration#component-name-casing) lorsque cela est possible, car il améliore la lisibilité du template en différenciant les composants Vue des éléments natifs. Cependant, il n'y a pas autant d'avantages pratiques à utiliser le camelCase lors du passage de props, nous avons donc choisi de suivre les conventions de chaque langage.

### Props statiques vs. dynamiques {#static-vs-dynamic-props}

Jusqu'à présent, vous avez vu des props passés en tant que valeurs statiques, comme dans :

```vue-html
<BlogPost title="My journey with Vue" />
```

Vous avez également vu des props assignés dynamiquement avec `v-bind` ou son raccourci `:`, comme dans :

```vue-html
<!-- Attribuer dynamiquement la valeur d'une variable -->
<BlogPost :title="post.title" />

<!-- Attribuer dynamiquement la valeur d'une expression complexe -->
<BlogPost :title="post.title + ' by ' + post.author.name" />
```

### Passage de différents types de valeur {#passing-different-value-types}

Dans les deux exemples ci-dessus, nous passons des chaînes de caractères, mais _n'importe quel_ type de valeur peut être passé à un prop.

#### Nombre {#number}

```vue-html
<!-- Même si `42` est statique, nous avons besoin de v-bind pour indiquer à Vue qu'il -->
<!-- s'agit d'une expression JavaScript plutôt que d'une chaîne de caractères.        -->
<BlogPost :likes="42" />

<!-- Attribuer dynamiquement la valeur d'une variable. -->
<BlogPost :likes="post.likes" />
```

#### Booléen {#boolean}

```vue-html
<!-- Inclure la prop sans valeur impliquera `true`. -->
<BlogPost is-published />

<!-- Même si `false` est statique, nous avons besoin de v-bind pour indiquer à Vue qu'il -->
<!-- s'agit d'une expression JavaScript plutôt que d'une chaîne de caractères.           -->
<BlogPost :is-published="false" />

<!-- Attribuer dynamiquement la valeur d'une variable. -->
<BlogPost :is-published="post.isPublished" />
```

#### Tableau {#array}

```vue-html
<!-- Même si le tableau est statique, nous avons besoin de v-bind pour indiquer à Vue qu'il -->
<!-- s'agit d'une expression JavaScript plutôt que d'une chaîne de caractères.              -->
<BlogPost :comment-ids="[234, 266, 273]" />

<!-- Attribuer dynamiquement la valeur d'une variable. -->
<BlogPost :comment-ids="post.commentIds" />
```

#### Objet {object}

```vue-html
<!-- Même si l'objet est statique, nous avons besoin de v-bind pour indiquer à Vue qu'il -->
<!-- s'agit d'une expression JavaScript plutôt que d'une chaîne de caractères.           -->
<BlogPost
  :author="{
    name: 'Veronica',
    company: 'Veridian Dynamics'
  }"
 />

<!-- Attribuer dynamiquement la valeur d'une variable. -->
<BlogPost :author="post.author" />
```

### Liaison de plusieurs propriétés à l'aide d'un objet {#binding-multiple-properties-using-an-object}

Si vous souhaitez transmettre toutes les propriétés d'un objet en tant que props, vous pouvez utiliser [`v-bind` sans argument](/guide/essentials/template-syntax#dynamically-binding-multiple-attributes) (`v-bind` au lieu de `:prop-name`). Par exemple, étant donné un objet `post` :

<div class="options-api">

```js
export default {
  data() {
    return {
      post: {
        id: 1,
        title: 'My Journey with Vue'
      }
    }
  }
}
```

</div>
<div class="composition-api">

```js
const post = {
  id: 1,
  title: 'My Journey with Vue'
}
```

</div>

Le template suivant :

```vue-html
<BlogPost v-bind="post" />
```

Sera équivalent à :

```vue-html
<BlogPost :id="post.id" :title="post.title" />
```

## Flux de données à sens unique {#one-way-data-flow}

Tous les props forment une **liaison unidirectionnelle** entre la propriété enfant et la propriété parent : lorsque la propriété parent est mise à jour, elle descendra vers l'enfant, mais pas l'inverse. Cela empêche les composants enfants de muter accidentellement l'état du parent, ce qui peut rendre le flux de données de votre application plus difficile à comprendre.

De plus, chaque fois que le composant parent est mis à jour, tous les props du composant enfant seront actualisés avec la dernière valeur. Cela signifie que vous **ne devez pas** tenter de muter une prop à l'intérieur d'un composant enfant. Si vous le faites, Vue vous avertira dans la console :

<div class="composition-api">

```js
const props = defineProps(['foo'])

// ❌ avertissement, les props sont en lecture seule !
props.foo = 'bar'
```

</div>
<div class="options-api">

```js
export default {
  props: ['foo'],
  created() {
    // ❌ avertissement, les props sont en lecture seule !
    this.foo = 'bar'
  }
}
```

</div>

Il y a généralement deux cas où il est tentant de muter une prop :

1. **Le prop est utilisé pour passer une valeur initiale; le composant enfant veut ensuite l'utiliser comme propriété de données locale.** Dans ce cas, il est préférable de définir une propriété de données locale qui utilise la prop comme valeur initiale :

   <div class="composition-api">

   ```js
   const props = defineProps(['initialCounter'])

   // counter utilise uniquement props.initialCounter comme valeur initiale ;
   // il est déconnecté des futures mises à jour de props.
   const counter = ref(props.initialCounter)
   ```

   </div>
   <div class="options-api">

   ```js
   export default {
     props: ['initialCounter'],
     data() {
       return {
         // counter utilise uniquement this.initialCounter comme valeur initiale ;
         // il est déconnecté des futures mises à jour de props.
         counter: this.initialCounter
       }
     }
   }
   ```

   </div>

2. **La prop est transmise en tant que valeur brute qui doit être transformée.** Dans ce cas, il est préférable de définir une propriété calculée à l'aide de la valeur de la prop :

   <div class="composition-api">

   ```js
   const props = defineProps(['size'])

   // propriété calculée qui se met à jour automatiquement lorsque la prop change
   const normalizedSize = computed(() => props.size.trim().toLowerCase())
   ```

   </div>
   <div class="options-api">

   ```js
   export default {
     props: ['size'],
     computed: {
       // propriété calculée qui se met à jour automatiquement lorsque la prop change
       normalizedSize() {
         return this.size.trim().toLowerCase()
       }
     }
   }
   ```

   </div>

### Mutation de props de type objet et tableau {#mutating-object-array-props}

Lorsque des objets et des tableaux sont passés en tant que props, bien que le composant enfant ne puisse pas muter la liaison de la prop, il **pourra** muter les propriétés imbriquées de l'objet ou du tableau. En effet, en JavaScript, les objets et les tableaux sont passés par référence, et il est déraisonnablement coûteux pour Vue d'empêcher de telles mutations.

Le principal inconvénient de ces mutations est qu'elles permettent au composant enfant de modifier l'état parent d'une manière qui n'est pas évidente vis-à-vis du composant parent, ce qui rend potentiellement plus difficile la compréhension du flux de données et sa maintenabilité dans le futur. En tant que bonne pratique, vous devez éviter de telles mutations à moins que le parent et l'enfant ne soient étroitement couplés par conception. Dans la plupart des cas, l'enfant doit [émettre un événement](/guide/components/events) pour laisser le parent effectuer la mutation.

## Validation de prop {#prop-validation}

Les composants peuvent spécifier des exigences pour leurs props, tels que la déclaration des types que vous avez déjà vus. Si une exigence n'est pas remplie, Vue vous avertira dans la console JavaScript du navigateur. Ceci est particulièrement utile lors du développement d'un composant destiné à être utilisé par d'autres.

Pour spécifier les validations de props, vous pouvez fournir un objet avec des exigences de validation à <span class="composition-api">la macro `defineProps()`</span><span class="options-api">l'option `props`</span>, au lieu d'un tableau de chaînes de caractères. Par exemple :

<div class="composition-api">

```js
defineProps({
  // Contrôle de type de base
  //  (les valeurs `null` et `undefined` autoriseront tous les types)
  propA: Number,
  // Plusieurs types possibles
  propB: [String, Number],
  // Chaîne de caractères requise
  propC: {
    type: String,
    required: true
  },
  // Nombre avec une valeur par défaut
  propD: {
    type: Number,
    default: 100
  },
  // Objet avec une valeur par défaut
  propE: {
    type: Object,
    // Les valeurs par défaut d'un objet ou d'un tableau doivent être renvoyées à partir
    // d'une fonction factory. La fonction reçoit les props bruts
    // reçus par le composant en tant qu'argument.
    default(rawProps) {
      return { message: 'hello' }
    }
  },
  // Fonction de validation personnalisée
  propF: {
    validator(value) {
      // La valeur doit correspondre à l'une de ces chaînes de caractères
      return ['success', 'warning', 'danger'].includes(value)
    }
  },
  // Fonction avec une valeur par défaut
  propG: {
    type: Function,
    // Contrairement aux valeurs par défaut d'un objet ou d'un tableau, il ne s'agit pas d'une fonction
    // factory - il s'agit d'une fonction servant de valeur par défaut
    default() {
      return 'Default function'
    }
  }
})
```

:::tip
Le code à l'intérieur de l'argument `defineProps()` **ne peut pas accéder aux autres variables déclarées dans `<script setup>`**, car l'expression entière est déplacée vers une portée de fonction externe lors de la compilation.
:::

</div>
<div class="options-api">

```js
export default {
  props: {
    // Contrôle de type de base
    //  (les valeurs`null` et `undefined` autoriseront tous les types)
    propA: Number,
    // Plusieurs types possibles
    propB: [String, Number],
    // Chaîne de caractères requise
    propC: {
      type: String,
      required: true
    },
    // Nombre avec une valeur par défaut
    propD: {
      type: Number,
      default: 100
    },
    // Objet avec une valeur par défaut
    propE: {
      type: Object,
      // Les valeurs par défaut d'un objet ou d'un tableau doivent être renvoyées à partir
      // d'une fonction factory. La fonction reçoit les props bruts
      // reçus par le composant en tant qu'argument.
      default(rawProps) {
        return { message: 'hello' }
      }
    },
    // Fonction de validation personnalisée
    propF: {
      validator(value) {
        // The value must match one of these strings
        return ['success', 'warning', 'danger'].includes(value)
      }
    },
    // Fonction avec une valeur par défaut
    propG: {
      type: Function,
      // Contrairement aux valeurs par défaut d'un objet ou d'un tableau, il ne s'agit pas d'une fonction
      // factory - il s'agit d'une fonction servant de valeur par défaut
      default() {
        return 'Default function'
      }
    }
  }
}
```

</div>

Détails supplémentaires :

- Toutes les props sont facultatives par défaut, sauf si `required: true` est spécifié.

- Une prop facultative absente autre que `Boolean` aura la valeur `undefined`.

- Les props `Boolean` absentes seront converties en `false`. Vous pouvez changer cela en définissant une valeur par défaut, c'est-à-dire: `default: undefined` pour se comporter comme une prop non booléen.

- Si une valeur `default` est spécifiée, elle sera utilisée si la valeur prop résolue est `undefined` - cela inclut à la fois lorsque la prop est absente ou lorsqu'une valeur `undefined` explicite est passée.

Lorsque la validation de prop échoue, Vue produira un avertissement dans la console (si vous utilisez la version de développement).

<div class="composition-api">

Si vous utilisez des [déclarations de props basées sur les types](/api/sfc-script-setup#type-only-props-emit-declarations) <sup class="vt-badge ts" />, Vue fera de son mieux pour compiler les annotations de type dans des déclarations de props équivalentes lors de l'exécution. Par exemple, `defineProps<{ msg: string }>` sera compilé en `{ msg: { type: String, required: true }}`.

</div>
<div class="options-api">

::: tip Note
Notez que les props sont validés **avant** qu'une instance de composant soit créée, de sorte que les propriétés d'instance (par exemple, `data`, `computed`, etc.) ne seront pas disponibles dans les fonctions `default` ou `validator`.
:::

</div>

### Vérifications de type à l'exécution {#runtime-type-checks}

Le `type` peut être l'un des constructeurs natifs suivants :

- `String`
- `Number`
- `Boolean`
- `Array`
- `Object`
- `Date`
- `Function`
- `Symbol`

De plus, `type` peut également être une classe personnalisée ou une fonction constructeur et l'assertion sera faite avec une vérification `instanceof`. Par exemple, étant donné la classe suivante :

```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
  }
}
```

Vous pouvez l'utiliser comme type de prop :

<div class="composition-api">

```js
defineProps({
  author: Person
})
```

</div>
<div class="options-api">

```js
export default {
  props: {
    author: Person
  }
}
```

</div>

Vue utilisera `instanceof Person` pour valider si la valeur de la prop `author` est bien une instance de la classe `Person`.

## Conversion en booléen {#boolean-casting}

Les props avec le type `Boolean` ont des règles de conversion spéciales pour imiter le comportement des attributs booléens natifs. Admettons un composant `<MyComponent>` avec la déclaration suivante :

<div class="composition-api">

```js
defineProps({
  disabled: Boolean
})
```

</div>
<div class="options-api">

```js
export default {
  props: {
    disabled: Boolean
  }
}
```

</div>

Le composant peut être utilisé comme ceci :

```vue-html
<!-- équivalent à passer :disabled="true" -->
<MyComponent disabled />

<!-- équivalent à passer :disabled="false" -->
<MyComponent />
```

Lorsqu'un accessoire est déclaré pour autoriser plusieurs types, les règles de conversion pour `Boolean` seront également appliquées. Cependant, il y a un avantage lorsque `String` et `Boolean` sont autorisés - la règle de casting sur Boolean ne s'applique que si Boolean apparaît avant String :

<div class="composition-api">

```js
// disabled sera transformé en true
defineProps({
  disabled: [Boolean, Number]
})

// disabled sera transformé en true
defineProps({
  disabled: [Boolean, String]
})

// disabled sera transformé en true
defineProps({
  disabled: [Number, Boolean]
})

// disabled sera analysé comme une chaîne vide (disabled="")
defineProps({
  disabled: [String, Boolean]
})
```

</div>
<div class="options-api">

```js
// disabled sera transformé en true
export default {
  props: {
    disabled: [Boolean, Number]
  }
}

// disabled sera transformé en true
export default {
  props: {
    disabled: [Boolean, String]
  }
}

// disabled sera transformé en true
export default {
  props: {
    disabled: [Number, Boolean]
  }
}

// disabled sera analysé comme une chaîne vide (disabled="")
export default {
  props: {
    disabled: [String, Boolean]
  }
}
```

</div>
