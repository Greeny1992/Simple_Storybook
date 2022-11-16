# Welcome to the Storybook Lab Session

In this session, we will build a UI from scratch in Angular. On the way to our UI we will walk through essential techniques.
This tutorial is based on the original Storybook intro at [Storybook](https://storybook.js.org/). When you have already done the online-course on moodle you can skip the first step!
Our goal at the end of the session, will be a simple task list, similar to this example here:

![Storybook example list](https://raw.githubusercontent.com/Greeny1992/Simple_Storybook/main/Simple_Storybook/assets/10-11-_2022_14-21-55.png)

## Table of Content

 1. [Get started](#get-started)
 2. [Simple component](#simple-component) 
 3. [Composite component](#composite-component)
 4. [Data](#data)
 5. [Screens](#screens)

----------------------
## Get started

To set up storybook, it is necessary, to already have a working angular app.
At first you have to run the following command in the existing project's root directory
```
# Add Storybook:
npx storybook init
```

The command above will make the following changes to your local env:

- Install all required dependencies
- Setup necessary scripts to run/build Storybook
- Add default Storybook config
- Add boilerplate stories to get you started
- Setup telemetry

The next step will be to start your Storybook with this command
```
npm run storybook 
```

----------------

## Simple component

To build our task list in a C-DD manner, we have to think of the simplest component of our list.
The simplest component will be a Task, which has a

- title ➡️ a string describing the task
- state ➡️ is the task checked (archived) or pinned

At first, let's create the component and the accompanying story file:
*src/app/components/task.component.ts*
*src/app/components/task.stories.ts*

```ts
// src/app/components/task.component.ts
import { Component, Input, Output, EventEmitter } from '@angular/core';

import { Task } from '../models/task.model';

@Component({
  selector: 'app-task',
  template: `
    <div class="list-item {{ task?.state }}">
      <label
        [attr.aria-label]="'archiveTask-' + task.id"
        for="checked-{{ task?.id }}"
        class="checkbox"
      >
        <input
          type="checkbox"
          disabled="true"
          [defaultChecked]="task?.state === 'TASK_ARCHIVED'"
          name="checked-{{ task?.id }}"
          id="checked-{{ task?.id }}"
        />
        <span class="checkbox-custom" (click)="onArchive(task.id)"></span>
      </label>
      <label
        [attr.aria-label]="task.title + ''"
        for="title-{{ task?.id }}"
        class="title"
      >
        <input
          type="text"
          [value]="task.title"
          readonly="true"
          id="title-{{ task?.id }}"
          name="title-{{ task?.id }}"
          placeholder="Input title"
        />
      </label>
      <button
        *ngIf="task?.state !== 'TASK_ARCHIVED'"
        class="pin-button"
        [attr.aria-label]="'pinTask-' + task.id"
        (click)="onPin(task.id)"
      >
        <span class="icon-star"></span>
      </button>
    </div>
  `,
})
export class TaskComponent {
  @Input() task: Task = {} as Task;

  // tslint:disable-next-line: no-output-on-prefix
  @Output()
  onPinTask = new EventEmitter<Event>();

  // tslint:disable-next-line: no-output-on-prefix
  @Output()
  onArchiveTask = new EventEmitter<Event>();

  /**
   * Component method to trigger the onPin event
   * @param id string
   */
  onPin(id: any) {
    this.onPinTask.emit(id);
  }
  /**
   * Component method to trigger the onArchive event
   * @param id string
   */
  onArchive(id: any) {
    this.onArchiveTask.emit(id);
  }
}

```

Now you have 10 minutes to write some tests for this component. Be creative and remember to the online session
<details>
  <summary>Click me for solution</summary>

```ts

//src/app/components/task.stories.ts
import { Meta, Story } from '@storybook/angular';

import { action } from '@storybook/addon-actions';

import { TaskComponent } from './task.component';

export default {
  component: TaskComponent,
  title: 'Task',
  excludeStories: /.*Data$/,
} as Meta;

export const actionsData = {
  onPinTask: action('onPinTask'),
  onArchiveTask: action('onArchiveTask'),
};

const Template: Story = args => ({
  props: {
    ...args,
    onPinTask: actionsData.onPinTask,
    onArchiveTask: actionsData.onArchiveTask,
  },
});

export const Default = Template.bind({});
Default.args = {
  task: {
    id: '1',
    title: 'Test Task',
    state: 'TASK_INBOX',
  },
};

export const Pinned = Template.bind({});
Pinned.args = {
  task: {
    ...Default.args['task'],
    state: 'TASK_PINNED',
  },
};

export const Archived = Template.bind({});
Archived.args = {
  task: {
    ...Default.args['task'],
    state: 'TASK_ARCHIVED',
  },
};
```

  </details>

---------------------

## Composite component

So the next step will be to build up a composite component. This means that we want to combine the componentent from the previous step together to a list. We do this to see what happens if we add more complexity.

The tasklist like it is implemented has a sort mechanism. That means that an pinned task is showed at the top of the list. Than we have also a loading screen and a empty state.

```ts
//src/app/components/task-list.component.ts
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { Task } from '../models/task.model';

@Component({
  selector: 'app-task-list',
 template: `
   <div class="list-items">
     <app-task
       *ngFor="let task of tasksInOrder"
       [task]="task"
       (onArchiveTask)="onArchiveTask.emit($event)"
       (onPinTask)="onPinTask.emit($event)"
     >
     </app-task>
     <div
       *ngIf="tasksInOrder.length === 0 && !loading"
       class="wrapper-message"
     >
       <span class="icon-check"></span>
       <p class="title-message">You have no tasks</p>
       <p class="subtitle-message">Sit back and relax</p>
     </div>
     <div *ngIf="loading">
       <div *ngFor="let i of [1, 2, 3, 4, 5, 6]" class="loading-item">
         <span class="glow-checkbox"></span>
         <span class="glow-text">
           <span>Loading</span> <span>cool</span> <span>state</span>
         </span>
       </div>
     </div>
   </div>
  `,
})
export class TaskListComponent {

  /**
  * @ignore
  * Component property to define ordering of tasks
  */
 tasksInOrder: Task[] = [];

  @Input() loading = false;

  // tslint:disable-next-line: no-output-on-prefix
  @Output() onPinTask: EventEmitter<any> = new EventEmitter();

  // tslint:disable-next-line: no-output-on-prefix
  @Output() onArchiveTask: EventEmitter<any> = new EventEmitter();

 @Input()
 set tasks(arr: Task[]) {
   const initialTasks = [
     ...arr.filter(t => t.state === 'TASK_PINNED'),
     ...arr.filter(t => t.state !== 'TASK_PINNED'),
   ];
   const filteredTasks = initialTasks.filter(
     t => t.state === 'TASK_INBOX' || t.state === 'TASK_PINNED'
   );
   this.tasksInOrder = filteredTasks.filter(
     t => t.state === 'TASK_INBOX' || t.state === 'TASK_PINNED'
   );
 }
}
```

Like before you have now 15 minutes to write some tests for this component. Be creative and remember to the online session. At the end there should be a minimum of four stories.

<details>
  <summary>Click me for solution</summary>

```ts
//src/app/components/task-list.stories.ts

import { componentWrapperDecorator, moduleMetadata, Meta, Story } from '@storybook/angular';

import { CommonModule } from '@angular/common';

import { TaskListComponent } from './task-list.component';
import { TaskComponent } from './task.component';

import * as TaskStories from './task.stories';

export default {
  component: TaskListComponent,
  decorators: [
    moduleMetadata({
      //👇 Imports both components to allow component composition with Storybook
      declarations: [TaskListComponent, TaskComponent],
      imports: [CommonModule],
    }),
    //👇 Wraps our stories with a decorator
    componentWrapperDecorator(story => `<div style="margin: 3em">${story}</div>`),
  ],
  title: 'TaskList',
} as Meta;

const Template: Story = args => ({
  props: {
    ...args,
    onPinTask: TaskStories.actionsData.onPinTask,
    onArchiveTask: TaskStories.actionsData.onArchiveTask,
  },
});


// 👇 This is the default story with basic data.
export const Default = Template.bind({});
Default.args = {
  tasks: [
    { ...TaskStories.Default.args?.['task'], id: '1', title: 'Task 1' },
    { ...TaskStories.Default.args?.['task'], id: '2', title: 'Task 2' },
    { ...TaskStories.Default.args?.['task'], id: '3', title: 'Task 3' },
    { ...TaskStories.Default.args?.['task'], id: '4', title: 'Task 4' },
    { ...TaskStories.Default.args?.['task'], id: '5', title: 'Task 5' },
    { ...TaskStories.Default.args?.['task'], id: '6', title: 'Task 6' },
  ],
};

// 👇 We extend the basic data to have one pinned task
export const WithPinnedTasks = Template.bind({});
WithPinnedTasks.args = {
  // Shaping the stories through args composition.
  // Inherited data coming from the Default story.
  tasks: [
    ...Default.args['tasks'].slice(0, 5),
    { id: '6', title: 'Task 6 (pinned)', state: 'TASK_PINNED' },
  ],
};

// 👇 Create a loading story.
export const Loading = Template.bind({});
Loading.args = {
  tasks: [],
  loading: true,
};


// 👇 Create a empty story.
export const Empty = Template.bind({});
Empty.args = {
  // Shaping the stories through args composition.
  // Inherited data coming from the Loading story.
  ...Loading.args,
  loading: false,
};
```

  </details>

