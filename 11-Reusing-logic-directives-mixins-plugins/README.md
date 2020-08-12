# Переиспользование логики

Часто требуется одну и ту же логику переиспользовать в разных Vue-компонентах.

Это могут быть одинаковые части описания компонента, общие функции, модификации и т.д.

На 7-ом вебинаре рассматривались способы глобального взаимодействия компонентов. Некоторые из них также проходят для переиспользования логики.
- Например, `provide/inject` позволяет из компонента предоставить функции всем компонентам потомкам. Так можно передать сервисы, функции доступа к специальным компонентам или общие данные.
- Патчинг прототипа Vue позволяет внедрить функцию или объект во все компоненты всех Vue приложений.
- Нельзя также забывать про самый базовый способ переиспользовать логику в JavaScript приложениях -- обычные ECMAScript модули.

Но всё это не позволяет добавить более сложные, Vue-специфические вещи.

## Директивы

Мы знакомы с множеством стандартных директив во Vue.js: `v-if`, `v-on`, `v-bind`. Vue также предоставляет возможность создавать свои директивы.

**Кастомные директивы**, как и все директивы во Vue.js могут принимать аргументы, модификаторы и значения или выражения, и влиять на работу компонента, на который они установлены.

Рассмотрим задачу. Требуется выделить содержимое поля ввода при фокусе. Таким образом, пользователь, если переходит на поле, может сразу ввести новое значение, не стирая старое (удобно в определённых ситуациях).

```vue
<input @focus="selectOnFocus" />
```

```javascript
selectOnFocus($event) {
  $event.currentTarget.setSelectionRange(0, -1);
}
```

Простое решение -- повесить обработчик события focus, который вызовет метод выделения.

А что, если тоже надо делать во многих компонентах?

Можно вынести эту функцию в отдельный модуль или иным образом импортировать там, где она требуется, и устанавливать обработчиком события. Но при таком решении мы знаем реализацию задачи и сами "дособираем" решение на месте. Некрасиво. А ведь это очень простая задача без параметров.

Решение -- создать свою директиву `v-select-on-focus`.

Директива описывается объектом с пятью хуками (все опциональны):
- `bind` -- вызывается один раз, когда директива привязывается к узлу (инициализация);
- `inserted` -- вызывается, когда элемент вставляется в DOM дерево (в родительский элемент);
- `update` -- вызывается, когда родительский компонент обновляется;
- `componentUpdated` -- вызывается после обновления родительского компонента и всех его потомков;  
- `unbind` -- вызывается, когда директива отвязывается от элемента (деинициализация).

Параметры каждого хука:
- `el` -- собственно элемент, на который повесили директиву;
- `binding` - объект с параметрами установки директивы (ниже);
- `vnode` - vnode элемента;
- `oldVnode` - старый vnode.
 
Параметр `binding` содержит параметры установки директивы, такие как:
- `name` -- имя;
- `value` -- значение;
- `oldValue` -- предыдущее значение;
- `expression` выражение;
- `arg` -- аргументы;
- `modifiers` -- модификаторы.
 
В нашей простой задаче требуется однократно при инициализации директивы добавить обработчик события `focus`. И не забыть его убрать при деинициализации, чтобы не получить утечку!

Описание директивы получится следующее.

```javascript
const handler = (e) => {
  e.currentTarget.setSelectionRange(0, -1);
};

const selectOnFocus = {
  bind(el) {
    el.addEventListener('focus', handler);
  },
  unbind(el) {
    el.addEventListener('focus', handler);
  },
};
```

Зарегистрировать директиву (как и, например, компонент) можно двумя способами: локально и глобально.

Глобально:
```javascript
Vue.directive('select-on-focus', selectOnFocus);
``` 

И локально:
```javascript
// Some component (or vue instance)
const SomeComponent = {
  template: '',
  components: '',

  directives: {
    selectOnFocus,   
  },

  data() {},
  // ...
};
```

После регистрации её можно будет использовать в шаблоне с префиксом `v-`.

```vue
<input v-select-on-focus />
```

Документация: [https://vuejs.org/v2/guide/custom-directive.html](https://vuejs.org/v2/guide/custom-directive.html).

## Миксины

**Миксины** (примеси) -- основной способ переиспользования логики по Vue.js (второй версии).

Миксин -- это объект, содержащий различные опции компонента в том же формате, в котором описывается компонент.

Регистрируя миксины, можно **примешивать** опции к компоненту, таким образом создавая компонент с помощью **композиции** (или дополняя его).

Это может быть что-то простое, например, миксин, который добавляет свойства `windowWidth` и `windowHeight` с размером окна в свойства компонента.

А может быть что-то более сложное, динамическое. Например, миксин, который генерирует локальную копию входного параметра.

Есть три способа регистрации миксинов:
1. Глобальная регистрация через `Vue.mixin(SomeMixin)` примешает опции ко всем компонентам;
2. Локальная регистрация через опцию `mixins: [mixin1, mixin2...]` примешает список миксинов к определённому компоненту;
3. Локальная регистрация через опцию `extends: baseMixin` аналогично предыдущему способу, но подразумевает, что компонент расширяет какой-то миксин.

Миксины можно сравнить с композицией или множественным наследованием, а `extends` - с обычным наследованием.

Тут стоит добавить цитату из документации React о наследовании компонентов.

> At Facebook, we use React in thousands of components, and we haven’t found any use cases where we would recommend creating component inheritance hierarchies.

Хотя миксины активно используются во множестве библиотек, их использование по мнению многих разработчиков -- плохая практика.

Основные недостатки миксинов:
- Конфликты имён. Если несколько миксинов имеют одинаковые опции компонента, возникнет конфликт;
- Неявность. Подключая миксин, мы не знаем, что именно он добавляет в компонент. Изменение компонентов ещё более неявное при использовании глобальных миксинов;
- Ещё запутаннее код становится, если миксины зависят от свойств друг-друга.

Во Vue 3 появляется новый основной способ переиспользования логики в компонентах -- [Composition API](https://composition-api.vuejs.org/).

Документация: [https://vuejs.org/v2/guide/mixins.html](https://vuejs.org/v2/guide/mixins.html).  

## Плагины

При разработке библиотек (и модулей приложения) требуется разным способом модифицировать Vue:
- Добавлять глобальные свойства или методы;
- Добавлять свойства и методы в прототип Vue;
- Глобально регистрировать компоненты, директивы, миксины, фильтры;
- Выполнять иные действия, необходимые для инициализации библиотеки.

Vue предлагает унифицированный способ регистрации библиотек -- **плагины**.

**Плагин** -- это объект с методом `install`, который принимает 2 параметра: `Vue` и параметры компонента. В этом методе выполняется всё, что требуется для инициализации плагина.

Для подключения плагина используется метод `Vue.use(Plugin, options)`.

Документация: [https://vuejs.org/v2/guide/plugins.html](https://vuejs.org/v2/guide/plugins.html).

## Фильтры

Во Vue 2 существует ещё один инструмент -- фильтры. Но они удалены во Vue 3.

Документация: [https://vuejs.org/v2/guide/filters.html](https://vuejs.org/v2/guide/filters.html).

RFC об удалении фильтров: [https://github.com/vuejs/rfcs/blob/master/active-rfcs/0015-remove-filters.md](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0015-remove-filters.md).

