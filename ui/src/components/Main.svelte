<script>
  import { onMount, afterUpdate } from 'svelte';
  import { DISPATCH, removeDispatch, RESPOND_KEYBIND, IS_RIGHT_MARGIN, shortCalls } from '@store/stores';
  import { fly } from 'svelte/transition';
  import { timeAgo } from '@utils/timeAgo';

  let notifications = [];
  
  DISPATCH.subscribe(value => {
    notifications = value || [];
  });

  function removeNotification(id) {
    removeDispatch(id);
  }

  onMount(() => {
    notifications.forEach(notification => {
      const { data, timer } = notification;
      setTimeout(() => {
        removeNotification(data.id);
      }, timer);
    });
  });

  afterUpdate(() => {
    notifications.forEach(notification => {
      const { data, timer } = notification;
      setTimeout(() => {
        removeNotification(data.id);
      }, timer);
    });
  });

  function getDispatchData(dispatch) {
    return [
      { icon: 'fas fa-map-marker-alt', value: dispatch.data.street },
      { icon: 'fas fa-clock', value: timeAgo(dispatch.data.time) },
      { label: 'Priority', value: dispatch.data.priority }
    ];
  }
</script>

<style>
  .notification {
    background-color: #650000; /* Update to match the desired background color */
    border-radius: 10px;
    padding: 10px;
    margin-bottom: 10px;
    color: white; /* Text color */
  }
  .notification-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .notification-content {
    display: flex;
    flex-direction: column;
    gap: 5px;
  }
  .notification-icon {
    margin-right: 10px;
  }
  .notification-priority {
    background-color: #7a0b0b; /* Update to match the priority color */
    border-radius: 5px;
    padding: 5px;
  }
</style>

<div class="w-screen h-screen flex justify-end { $IS_RIGHT_MARGIN ? 'flex-row' : 'flex-row-reverse' } items-end">
  <div class="w-[25%] h-[97%]"
       class:ml-[2vh]={!$IS_RIGHT_MARGIN}
       class:mr-[2vh]={$IS_RIGHT_MARGIN}
      >
    {#each notifications.slice().reverse() as dispatch, index (dispatch.data.id)}
      <div class="notification" transition:fly="{{ x: $IS_RIGHT_MARGIN ? 400 : -400 }}">
        <div class="notification-header">
          <div class="flex items-center">
            <i class="fas fa-bullhorn notification-icon"></i>
            <p>{dispatch.data.message}</p>
          </div>
          <div class="flex items-center">
            <p class="notification-priority">{dispatch.data.code}</p>
            <p class="notification-priority">#{dispatch.data.id}</p>
          </div>
        </div>
        <div class="notification-content">
          {#each getDispatchData(dispatch) as field}
            {#if field.value}
              <p>
                <i class={field.icon + ' notification-icon'}></i>
                {field.label ? field.label + ': ' : ''}{field.value}
              </p>
            {/if}
          {/each}
        </div>
      </div>
    {/each}
  </div>
</div>
