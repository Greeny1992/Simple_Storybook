# Welcome to the Storybook Lab Session

In this session, we will build a UI from scratch in Angular. On the way to our UI we will walk through essential techniques.
This tutorial is based on the original Storybook intro at [Storybook](https://storybook.js.org/). When you have already done the online-course on moodle you can skip the first step!
Our goal at the end of the session, will be a simple task list, similar to this example here:

![Storybook example list](https://raw.githubusercontent.com/Greeny1992/Simple_Storybook/main/Simple_Storybook/assets/10-11-_2022_14-21-55.png)

## Table of Content

 1. [Get started](### Get started)
 2. [Simple component](### Simple component) 
 3. [Composite component](### Composite component)
 4. [Data](### Data)
 5. [Screens](### Screens)

### Get started

To set up storybook, it is necessary, to already have a working angular app.
At first you have to run the following command in the existing project's root directory

    # Add Storybook:
    npx storybook init

The command above will make the following changes to your local env:

- Install all required dependencies
- Setup necessary scripts to run/build Storybook
- Add default Storybook config
- Add boilerplate stories to get you started
- Setup telemetry

The next step will be to start your Storybook with this command

  npm run storybook 
----------------

### Simple component

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

