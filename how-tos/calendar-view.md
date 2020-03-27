# Calendar view

This example shows you how to use a smart view to visualize your data on a clalendar.

![](../.gitbook/assets/calendar-view.gif)



## Requirements

* [fullcalendar](https://fullcalendar.io/docs) integration \(4.2.0\)

## How it works

### File: template.hbs

This file contains handlebars and HTML declaration.

{% code title="template.hbs" %}
```css
<style>
  .calendar {
    padding: 20px;
    background: white;
    height: 100%;
    overflow: scroll;
  }
  .calendar .fc-toolbar.fc-header-toolbar .fc-left {
    font-size: 14px;
    font-weight: bold;
  }
  .calendar .fc-day-header {
    padding: 10px 0;
    background-color: #f7f7f7;
  }
  .calendar .fc-event {
    background-color: #f7f7f7;
    border: 1px solid #ddd;
    color: #555;
    font-size: 14px;
  }
  .calendar .fc-day-grid-event {
    background-color: #4285f4;
    color: white;
    font-size: 10px;
    border: none;
    padding: 2px;
  }
  .calendar .fc-time-grid-event {
    background-color: #4285f4;
    color: white;
    font-size: 10px;
    border: none;
    padding: 2px;
  }
  .popper,
  .tooltip {
    position: absolute;
    z-index: 9999;
    background: #71ba84;
    color: white;
    font-weight: bold;
    width: 150px;
    border-radius: 3px;
    box-shadow: 0 0 2px rgba(0, 0, 0, 0.5);
    padding: 10px;
    text-align: center;
  }
  .style5 .tooltip {
    background: #1E252B;
    color: #FFFFFF;
    max-width: 200px;
    width: auto;
    font-size: .8rem;
    padding: .5em 1em;
  }
  .popper .popper__arrow,
  .tooltip .tooltip-arrow {
    width: 0;
    height: 0;
    border-style: solid;
    position: absolute;
    margin: 5px;
  }
  .tooltip .tooltip-arrow,
  .popper .popper__arrow {
    border-color: #71ba84;
  }
  .style5 .tooltip .tooltip-arrow {
    border-color: #1E252B;
  }
  .popper[x-placement^="top"],
  .tooltip[x-placement^="top"] {
    margin-bottom: 5px;
  }
  .popper[x-placement^="top"] .popper__arrow,
  .tooltip[x-placement^="top"] .tooltip-arrow {
    border-width: 5px 5px 0 5px;
    border-left-color: transparent;
    border-right-color: transparent;
    border-bottom-color: transparent;
    bottom: -5px;
    left: calc(50% - 5px);
    margin-top: 0;
    margin-bottom: 0;
  }
  .popper[x-placement^="bottom"],
  .tooltip[x-placement^="bottom"] {
    margin-top: 5px;
  }
  .tooltip[x-placement^="bottom"] .tooltip-arrow,
  .popper[x-placement^="bottom"] .popper__arrow {
    border-width: 0 5px 5px 5px;
    border-left-color: transparent;
    border-right-color: transparent;
    border-top-color: transparent;
    top: -5px;
    left: calc(50% - 5px);
    margin-top: 0;
    margin-bottom: 0;
  }
  .tooltip[x-placement^="right"],
  .popper[x-placement^="right"] {
    margin-left: 5px;
  }
  .popper[x-placement^="right"] .popper__arrow,
  .tooltip[x-placement^="right"] .tooltip-arrow {
    border-width: 5px 5px 5px 0;
    border-left-color: transparent;
    border-top-color: transparent;
    border-bottom-color: transparent;
    left: -5px;
    top: calc(50% - 5px);
    margin-left: 0;
    margin-right: 0;
  }
  .popper[x-placement^="left"],
  .tooltip[x-placement^="left"] {
    margin-right: 5px;
  }
  .popper[x-placement^="left"] .popper__arrow,
  .tooltip[x-placement^="left"] .tooltip-arrow {
    border-width: 5px 0 5px 5px;
    border-top-color: transparent;
    border-right-color: transparent;
    border-bottom-color: transparent;
    right: -5px;
    top: calc(50% - 5px);
    margin-left: 0;
    margin-right: 0;
  }
</style>

<div id="{{calendarId}}" class="calendar"></div>
```
{% endcode %}

### File: javascript.js

This file handle all events or actions

{% code title="component.js" %}
```javascript
'use strict';
import Ember from 'ember';
import SmartViewMixin from 'client/mixins/smart-view-mixin';

let calendar = null;

export default Ember.Component.extend(SmartViewMixin.default, {
  store: Ember.inject.service(),
  conditionAfter: null,
  conditionBefore: null,
  loaded: false,
  calendarId: null,
  tooltipId: null,
  loadPlugin: function() {
    const that = this;
    Ember.run.scheduleOnce('afterRender', this, function() {
      if (this.get('viewList.recordPerPage') !== 50) {
        this.set('viewList.recordPerPage', 50);
        this.sendAction('updateRecordPerPage');
      }
      that.set('calendarId', `${this.get('element.id')}-calendar`);
      that.set('tooltipId', `${this.get('element.id')}-tooltip`);
      Ember.$.getScript('//unpkg.com/popper.js/dist/umd/popper.min.js', function() {
        Ember.$.getScript('//unpkg.com/tooltip.js/dist/umd/tooltip.min.js', function() {
          Ember.$.getScript(
            '//cdnjs.cloudflare.com/ajax/libs/fullcalendar/4.2.0/core/main.min.js',
            function() {
              Ember.$.getScript(
                '//cdnjs.cloudflare.com/ajax/libs/fullcalendar/4.2.0/daygrid/main.min.js',
                function() {
                  Ember.$.getScript(
                    '//cdnjs.cloudflare.com/ajax/libs/fullcalendar/4.2.0/timegrid/main.js',
                    function() {
                      const calendarEl = document.getElementById(that.get('calendarId'));

                      calendar = new FullCalendar.Calendar(calendarEl, {
                        header: {
                          left: 'title',
                          center: '',
                          right: 'today prev,next dayGridMonth,timeGridWeek',
                        },
                        allDaySlot: false,
                        plugins: ['dayGrid', 'timeGrid'],
                        defaultView: 'timeGridWeek',
                        defaultDate: new Date(),
                        viewRender: function(view, element) {
                          const field = that.get('collection.fields').findBy('field', 'datetime');

                          that.sendAction('fetchRecords', { page: 1 });
                        },
                        eventRender: function(eventData) {
                          new Tooltip(eventData.el, {
                            title: eventData.event.extendedProps.description,
                            placement: 'top',
                            trigger: 'hover',
                            container: 'body',
                          });
                        },
                        eventClick: function(eventData) {
                          that
                            .get('router')
                            .transitionTo(
                              'rendering.data.collection.list.viewEdit.details',
                              that.get('collection.id'),
                              eventData.event.id,
                            );
                        },
                      });

                      calendar.render();

                      that.set('loaded', true);
                    },
                  );
                },
              );
            },
          );
        });
      });

      const cssCoreLink = $('<link>');
      const cssDayGridLink = $('<link>');
      const cssTimegridLink = $('<link>');
      $('head').append(cssCoreLink);
      $('head').append(cssDayGridLink);
      $('head').append(cssTimegridLink);

      cssCoreLink.attr({
        rel: 'stylesheet',
        type: 'text/css',
        href: 'https://cdnjs.cloudflare.com/ajax/libs/fullcalendar/4.2.0/core/main.min.css',
      });
      cssDayGridLink.attr({
        rel: 'stylesheet',
        type: 'text/css',
        href: 'https://cdnjs.cloudflare.com/ajax/libs/fullcalendar/4.2.0/daygrid/main.min.css',
      });
      cssTimegridLink.attr({
        rel: 'stylesheet',
        type: 'text/css',
        href: 'https://cdnjs.cloudflare.com/ajax/libs/fullcalendar/4.2.0/timegrid/main.min.css',
      });
    });
  }.on('init'),
  setEvent: function() {
    if (!this.get('records')) {
      return;
    }

    this.get('records').forEach(function(deliverySlip) {
      const association = deliverySlip.get('forest-_associationId');
      const shop = deliverySlip.get('forest-_shopId');

      const event = {
        id: deliverySlip.get('id'),
        title: `${shop.get('forest-name')} -> ${association.get('forest-name')}`,
        description: `${shop.get('forest-name')} -> ${association.get('forest-name')}`,
        start: deliverySlip.get('forest-datetime'),
      };

      calendar.addEvent(event);
    });
  }.observes('loaded', 'records.[]'),
});
```
{% endcode %}

