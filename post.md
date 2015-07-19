Animating elements between parts of an application can be tricky. The beauty of `ng-repeat` means we can declare that a list, such as a `ul`, should represent a data model, such as an `Array` of data, and the list keeps up to date with whatever changes we make to the underlying model. The tricky bit comes when we want to view not just current state of the model, but transitions between states, such as 

- removal of an item from a list,
- addition of an item to a list, or
- moving an item from one list to another.

Prior to Angular 1.4, it was fairly straightforward to animate addition or removal of items from an `ng-repeat` powered list using `.ng-enter` and `.ng-leave` transitions, but there wasn't anything built in to animate items moving from one list to another. Now with 1.4, we can do exactly that.

The key to this is the `ng-animate-ref` attribute. If the ref-value on an _outgoing_ element matches an element on an _incoming_ element, then Angular clones the outgoing element, inserts the clone positioned absolutely on the page, and using CSS transitions moves it from the position of the outgoing to the position of the incoming.

Outgoing and incoming transitions
---------------------------------

What _outgoing_ and _incoming_ mean is that their addition to and removal from the DOM are subject to `.ng-enter` and `.ng-leave` transitions, either on themselves, or on a parent element. For example, we can animate the heights of elements to or from 0 when they are added or removed from a list:

```html
<ul class="list">
  <li class="item" ng-repeat="item in listA" ng-click="moveToListB(item)">
    Item: {{ item.id }}
  </li>
</ul>
<ul class="list">
  <li class="item" ng-repeat="item in listB" ng-click="moveToListA(item)">
    Item: {{ item.id }}
  </li>
</ul>
```

where the `moveToListX` functions move an item from one list to the other

```javascript
$scope.moveToListB = function(item) {
  $scope.listB.push(item);
  $scope.listA.splice($scope.listA.indexOf(item), 1);
};
```

and where the CSS transitions are defined as:

```css
/* New element set to 0 height after addition to DOM... */
.item.ng-enter {
  transition: 0.1s linear all;
  height: 0;
}

/* ...then transitioned to 30px */
.item.ng-enter.ng-enter-active {
  height: 30px;
}

/* Existing element set to 30px height just before removal... */
.item.ng-leave {
  transition: 0.1s linear all;
  height: 30px;
}

/* ... then transitioned to 0 */
.item.ng-leave.ng-leave-active {
  height: 0;
}
```

The above rules work because at the appropriate points in the addition and removal of elements from `ng-repeat` lists, Angular adds the classes on the elements and allows the browser to perform the CSS transitions. You can look into the [ngAnimate docs for CSS transitions](https://docs.angularjs.org/api/ngAnimate#css-based-animations) for more information on these.

Add ng-animate-ref attribute on wrapper elements
------------------------------------------------

Once the list elements are subject to `.ng-enter` and `.ng-leave` transitions, we add a wrapper `span` to each element, with an `ng-animate-ref` attribute containing the ID of the item, so Angular can match each element leaving the DOM with one entering.

```html
<ul class="list" title="List A">
  <li class="item" ng-repeat="item in listA" ng-click="moveToListB(item)">
    <span class="item-contents" ng-animate-ref="{{ item.id }}">Item: {{ item.id }}</span>
  </li>
</ul>
<ul class="list" title="List B">
  <li class="item" ng-repeat="item in listB" ng-click="moveToListA(item)">
    <span class="item-contents" ng-animate-ref="{{ item.id }}">Item: {{ item.id }}</span>
  </li>
</ul>
```

Note that depending on the transitions we want, we might not need the extra wrapping `span`, but it keeps the code as flexible as possible, and avoids any ambiguity about what transitions are happening on what elements: the enter and leave transitions take place on the `.item` elements, and the animations between the lists happen on the `.item-contents` elements.

We can control the CSS transition with a single style rule on the `.ng-anchor-in` class that Angular adds to the cloned element.

```css
.item-contents.ng-anchor-in {
  transition: 0.2s linear all;
}
```

You can have a more complex transition if you want by also having a transition on `.ng-anchor-out`, but this is not necessary for this case. You can see the [docs for ngAnimate](https://docs.angularjs.org/api/ngAnimate) for more information.

Plunker
-------

You can [see this in-action in this Plunker](http://plnkr.co/edit/L0XJS3vEZOACgKqtnqF3?p=preview) together with the the rest of the boilerplate Javascript and CSS.

That's it! There isn't really much to it.
