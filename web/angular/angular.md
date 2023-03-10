Objective: Summarize more advanced angular design patterns and utilities


# Performance

## [*ngFor] trackBy

> https://angular.io/api/core/TrackByFunction

When a list of objects update in the component, `*ngFor` will delete the HTML template for all objects in the browser DOM, and then paste in the new templates. 


```html
  <div *ngFor="let obj of myObjectList">
    {{ obj.name }}
  </div>
```

```typescript
export class MyComponent {
  public myObjectList: MyObject[] = [{ id: 1, name: 'Bob' }];

  public updateList(): void {
    this.myObjectList = [{ id: 1, name: 'Joe' }];
  }
}
```

> When updateList() is invoked, the browser will delete all of the divs in the list and then insert them as before.

This is especially concerning when using the redux pattern.

```html
  <div *ngFor="let obj of (myObjectList$ | async)"></div>
```

> Every time the slice of state updates, it is reassigning the entire list.

---

TrackBy allows the DOM items to be looked up by a specific object property, so the browser doesn't have to destroy the list and do a full repaint of all objects. The object ID becomes the object reference.

The child template now will use the object reference and only will repaint the fields that changed, even if the list is replaced.


```html
  <div *ngFor="let obj of (myObjectList$ | async); trackBy identifyFn">
    {{ obj.name }}
  </div>
```

```typescript
export class MyComponent {
  public myObjectList$: BehaviorSubject<MyObject[]> = new BehaviorSubject([{ id: 1, name: 'Bob' }]);

  public updateList(): void {
    myObjectList$.next([{ id: 1, name: 'Joe' }]);
  }

  public identifyFn(index: number, item: MyObject): void {
    return item.id;
  }
}
```
> The div will not be removed, instead, only `obj.name` with `id = 1` will perform a DOM update. Way more efficient!

Another bonus is handling focus states. When the DOM element is destroyed and recreated, then it will lose focus after the click event. This is a great solution for a list of buttons.


This does not apply when mutating a list of objects directly. This will update the child templates as expected.
```typescript
myObjectList[0].name = 'Joe';
```

### Usage Tips

- The `identify` function can be named anything.
- Each call to the identify function should be a pure function accessor. Do not add additional logic as it is called for every time the list is updated for each item.
