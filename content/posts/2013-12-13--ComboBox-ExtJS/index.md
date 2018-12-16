---
title: New Plugin for checkcombo in Sencha ExtJs 4.2.2
category: ExtJS
tags: [javascript, ExtJS]
cover: answered.jpg
---
![Plugins](answered.jpg)

ExtJS does not come with mutl-check dropdown.I have create two new plugins.

## Ext.ux.PerCheckCombo

This plugin will provide checkcombo ( Checkbox as a list in combobox ) which have additional featute of `disableFieldFeature`.

**Features**:
- Multiselect using checkboxes
- Can be used in a form
- Checkbox can be disable/enable using `disableFieldFeature: true`

If `disableFieldFeature = false` then it will work like checkcombo or `disableFieldFeature = true` then it will have checkcombo with perticular checbox disable.

To access `disableFieldFeature` you should have to specify extra `disablefield` in store (shown in below example).

Example:
```javascript
// Store
let test_store = Ext.create('Ext.data.Store', {
  autoLoad: false,
  id: 'test_store_id',
  fields: [
    {
      name: 'id',
      type: 'string'
    },
    {
      name: 'name',
      type: 'string'
    },
    {
      name: 'disablefield',
      type: 'string'
    }
  ],
  proxy: {
    type: 'ajax',
    url: 'test.php',
    reader: {
      type: 'json',
      root: 'rows'
    }
  }
});
```
Usage:
```javascript
{
    xtype: 'persearchcheckcombo',
    name: 'test_name',
    fieldLabel: 'test',
    valueField: 'id',
    displayField: 'name',
    store: test_store,
    disableFieldFeature: true
}
```

## Ext.ux.PerSearchCheckCombo

This plugin will provide checkcombo ( checkbox as a list in combobox ) and search field which have extra feature of `disableFieldFeature`.

**Features**
  - Adds a search field in a checkcombo to allow user to filter the list
  - Can be used in a form
  - Checbox can be disable/enable based on `disableFieldFeature: true`

If `disableFieldFeature = false` then it will work like checkcombo or `disableFieldFeature = true` then it will have checkcombo with perticular checbox disable.

To access `disableFieldFeature` you should have to specify extra `disablefield` in store (shown in example).

Example:
```javascript
// Store
 var test_store = Ext.create('Ext.data.Store', {
  autoLoad: false,
  id: 'test_store_id',
  fields: [
    { name: 'id', type: 'string' },
    { name: 'name', type: 'string' },
    { name: 'disablefield', type: 'string' }
  ],
  proxy: {
    type: 'ajax',
    url: 'test.php',
    reader: { type: 'json', root: 'rows' }
  }
});
```
Usage
```javascript
{
  xtype: 'persearchcheckcombo',
  name: 'test_name',
  fieldLabel: 'test',
  valueField: 'id',
  displayField: 'name',
  store: test_store,
  disableFieldFeature: true
}
```

For More info check [this](https://www.sencha.com/forum/showthread.php?295648-Ext.ux.PerCheckCombo&p=1079358&viewfull=1#post1079358) thread on sencha community forum.

Check [this](https://www.sencha.com/forum/showthread.php?296637-Ext.ux.grid.filter.DateTimeFilter&p=1083191#post1083191) Date time filter plugin with lot of features.

# Plugin
### xtype : percheckcombo

```javascript
Ext.define('Ext.ux.CheckCombo',
{

    extend: 'Ext.form.field.ComboBox',
    alias: 'widget.checkcombo',
    multiSelect: true,
    allSelector: false,
    noData: false,
    noDataText: '',
    addAllSelector: false,
    allSelectorHidden: false,
    enableKeyEvents: true,
    afterExpandCheck: false,
    allText: 'All',
    oldValue: '',
    //matchFieldWidth: true,
    //listConfig: {minWidth: 50},
    listeners:
    {
/* uncomment if you want to reload store on every combo expand
        beforequery: function(qe)
        {
            this.store.removeAll();
            delete qe.combo.lastQuery;
        },
*/
        focus: function(cpt)
        {

            cpt.oldValue = cpt.getValue();
        },
        keydown: function(cpt, e, eOpts)
        {
            var    value = cpt.getRawValue(),
                oldValue = cpt.oldValue;
        }
    },
    createPicker: function()
    {
        var    me = this,
            picker,
            menuCls = Ext.baseCSSPrefix + 'menu',
            opts = Ext.apply(
            {
                pickerField: me,
                selModel:
                {
                    mode: me.multiSelect ? 'SIMPLE' : 'SINGLE'
                },
                floating: true,
                hidden: true,
                ownerCt: me.ownerCt,
                cls: me.el.up('.' + menuCls) ? menuCls : '',
                store: me.store,
                displayField: me.displayField,
                focusOnToFront: false,
                pageSize: me.pageSize,
                tpl:
                [
                    '<ul><tpl for=".">',
                        '<li role="option"   class="' + Ext.baseCSSPrefix + 'boundlist-item"><span class="x-combo-checker">&nbsp;</span> {' + me.displayField + '}</li>',
                    '</tpl></ul>'
                ]
            }, me.listConfig, me.defaultListConfig);


        picker = me.picker = Ext.create('Ext.view.BoundList', opts);
        if(me.pageSize)
        {
            picker.pagingToolbar.on('beforechange', me.onPageChange, me);
        }


        me.mon(picker,
        {
            itemclick: me.onItemClick,
            refresh: me.onListRefresh,
            scope: me
        });


        me.mon(picker.getSelectionModel(),
        {
            'beforeselect': me.onBeforeSelect,
            'beforedeselect': me.onBeforeDeselect,
            'selectionchange': me.onListSelectionChange,
            scope: me
        });


        me.store.on('load', function(store)
        {
            if(store.getTotalCount() == 0)
            {
                me.allSelectorHidden = true;
                if(me.allSelector != false) me.allSelector.setStyle('display', 'none');
                if(me.noData != false) me.noData.setStyle('display', 'block');
            }
            else
            {
                me.allSelectorHidden = false;
                if(me.allSelector != false) me.allSelector.setStyle('display', 'block');
                if(me.noData != false) me.noData.setStyle('display', 'none');
            }
        });
        return picker;
    },
    reset: function()
    {
        var    me = this;
        me.setValue('');
    },
    setValue: function(value)
    {
        this.value = value;
        if(!value)
        {
            if(this.allSelector != false) this.allSelector.removeCls('x-boundlist-selected');
            return this.callParent(arguments);
        }

        if(typeof value == 'string')
        {
            var    me = this,
                records = [],
                vals = value.split(',');

            if(value == '')
            {
                if(me.allSelector != false) me.allSelector.removeCls('x-boundlist-selected');
            }
            else
            {
                if(vals.length == me.store.getCount() && vals.length != 0)
                {
                    if(me.allSelector != false) me.allSelector.addCls('x-boundlist-selected');
                    else me.afterExpandCheck = true;
                }
            }

            Ext.each(vals, function(val)
            {
                var record = me.store.getById(parseInt(val));
                if(record) records.push(record);
            });
            return me.setValue(records);
        }
        else return this.callParent(arguments);
    },
    getValue: function()
    {
        if(typeof this.value == 'object') return this.value.join(',');
        else return this.value;
    },
    getSubmitValue: function()
    {
        return this.getValue();
    },
    expand: function()
    {
        var    me = this,
            bodyEl, picker, collapseIf;

            if(me.rendered && !me.isExpanded && !me.isDestroyed)
            {
            bodyEl = me.bodyEl;
            picker = me.getPicker();
            collapseIf = me.collapseIf;


            // show the picker and set isExpanded flag
            picker.show();
            me.isExpanded = true;
            me.alignPicker();
            bodyEl.addCls(me.openCls);

            if(me.noData == false) me.noData = picker.getEl().down('.x-boundlist-list-ct').insertHtml('beforeBegin', '<div class="x-boundlist-item" role="option">'+me.noDataText+'</div>', true);
            if(me.addAllSelector == true && me.allSelector == false)
            {
                var selectedvalues =  this.value ;
                  var vals =  String(selectedvalues).split(",");
                if(vals.length == me.store.getCount() && vals.length != 0)
                {
                me.allSelector = picker.getEl().down('.x-boundlist-list-ct').insertHtml('beforeBegin', '<div class="x-boundlist-item x-boundlist-selected" role="option"><span class="x-combo-checker">&nbsp;</span> '+me.allText+'</div>', true);
                }
                else
                {
                 me.allSelector = picker.getEl().down('.x-boundlist-list-ct').insertHtml('beforeBegin', '<div class="x-boundlist-item" role="option"><span class="x-combo-checker">&nbsp;</span> '+me.allText+'</div>', true);
                }
                me.allSelector.on('click', function(e)
                {
                    if(me.allSelector.hasCls('x-boundlist-selected'))
                    {
                        me.allSelector.removeCls('x-boundlist-selected');
                        me.setValue('');
                        me.fireEvent('select', me, []);
                    }
                    else
                    {
                        var records = [];
                        me.store.each(function(record)
                        {
                            records.push(record);
                        });
                        me.allSelector.addCls('x-boundlist-selected');
                        me.select(records);
                        me.fireEvent('select', me, records);
                    }
                });


                if(me.allSelectorHidden == true) me.allSelector.hide();
                else me.allSelector.show();

                if(me.afterExpandCheck == true)
                {
                    me.allSelector.addCls('x-boundlist-selected');
                    me.afterExpandCheck = false;
                }
            }

            // monitor clicking and mousewheel
            me.mon(Ext.getDoc(),
            {
                mousewheel: collapseIf,
                mousedown: collapseIf,
                scope: me
            });
            Ext.EventManager.onWindowResize(me.alignPicker, me);
            me.fireEvent('expand', me);
            me.onExpand();
        }
        else
        {
            me.fireEvent('expand', me);
            me.onExpand();
        }
    },
    alignPicker: function()
    {
        var me = this,
            picker = me.getPicker();


        me.callParent();

        if(me.addAllSelector == true)
        {
            var height = picker.getHeight();
            //alert(height);
            height = parseInt(height)+25;
            picker.setHeight(height);
           picker.getEl().setStyle('height', height+'px');
        }
    },
    onListSelectionChange: function(list, selectedRecords)
    {
        var    me = this,
            isMulti = me.multiSelect,
            hasRecords = selectedRecords.length > 0;
        // Only react to selection if it is not called from setValue, and if our list is
        // expanded (ignores changes to the selection model triggered elsewhere)
        if(!me.ignoreSelection && me.isExpanded)
        {
            if(!isMulti)
            {
                Ext.defer(me.collapse, 1, me);
            }
            /*
            * Only set the value here if we're in multi selection mode or we have
            * a selection. Otherwise setValue will be called with an empty value
            * which will cause the change event to fire twice.
            */
            if(isMulti || hasRecords)
            {
                me.setValue(selectedRecords, false);
            }
            if(hasRecords)
            {
                me.fireEvent('select', me, selectedRecords);
            }
            me.inputEl.focus();

            if(me.addAllSelector == true && me.allSelector != false)
            {
                if(selectedRecords.length == me.store.getTotalCount()) me.allSelector.addCls('x-boundlist-selected');
                else me.allSelector.removeCls('x-boundlist-selected');
            }
        }
    }
});
```

```javascript
Ext.define('Ext.ux.PerCheckCombo',{
    extend: 'Ext.ux.CheckCombo',
    alias: 'widget.percheckcombo',
    frame: false,
    createPicker: function()
    {
        var me = this,
        picker,
        menuCls = Ext.baseCSSPrefix + 'menu',
        opts = Ext.apply(
        {
            pickerField: me,
            selModel:
            {
                mode: me.multiSelect ? 'SIMPLE' : 'SINGLE'
            },
            floating: true,
            hidden: true,
            ownerCt: me.ownerCt,
            cls: me.el.up('.' + menuCls) ? menuCls : '',
            store: me.store,
            displayField: me.displayField,
            focusOnToFront: false,
            pageSize: me.pageSize,
            disableFieldFeature: false,
            bind: {
                store: this.store
            },
             listeners: {
                    beforeselect: function(combo, record, index ) {
                        if (record.get(me.disablefield) != 0) {
                            // combo.setValue(record.get(me.displayField));
                            // combo.setValue(record.get(me.displayField));
                            // console.log(combo);
                        } else {
                            return false;
                        }
                        // Cancel the default action
                    }

                },

            tpl:
            [
                   '<tpl if="'+me.disableFieldFeature+' == '+true+'">',
                        '<tpl><ul><tpl for=".">',
                            '<tpl if="disablefield != \'0\' ">',
                                '<li role="option"  class="' + Ext.baseCSSPrefix + 'boundlist-item"><span class="x-combo-checker">&nbsp;</span> {' + me.displayField  + '}</li>',
                            '<tpl else>',
                                '<li role="option"  class="' + Ext.baseCSSPrefix + 'boundlist-item" ><span><input type="checkbox" disabled="disabled" />&nbsp;</span> {' + me.displayField +'}</li>',
                            '</tpl>',
                        '</tpl></ul></tpl>' ,
                    '<tpl else>',
                      '<tpl><ul><tpl for=".">',
                            '<li role="option"  class="' + Ext.baseCSSPrefix + 'boundlist-item"><span class="x-combo-checker">&nbsp;</span> {' + me.displayField  + '}</li>',
                        '</tpl></ul></tpl>' ,
                     '</tpl>'

            ]
        }, me.listConfig, me.defaultListConfig);


        picker = me.picker = Ext.create('Ext.view.BoundList', opts);
        if(me.pageSize)
        {
            picker.pagingToolbar.on('beforechange', me.onPageChange, me);
        }

        me.mon(picker,
        {
            itemclick: me.onItemClick,
            refresh: me.onListRefresh,
            scope: me
        });

        me.mon(picker.getSelectionModel(),
        {
            'beforeselect': me.onBeforeSelect,
            'beforedeselect': me.onBeforeDeselect,
            'selectionchange': me.onListSelectionChange,
            scope: me
        });

        me.store.on('load', function(store)
        {
            if(store.getTotalCount() == 0)
            {
                me.allSelectorHidden = true;
                if(me.allSelector != false) me.allSelector.setStyle('display', 'none');
                if(me.noData != false) me.noData.setStyle('display', 'block');
            }
            else
            {
                me.allSelectorHidden = false;
                if(me.allSelector != false) me.allSelector.setStyle('display', 'block');
                if(me.noData != false) me.noData.setStyle('display', 'none');
            }
        });

        return picker;
    },

    setValue: function(value)
    {

        //Filter values based on disable condition
        if(typeof value != 'undefined' && this.disableFieldFeature === true)
        {

                var finalValue = [];
            if(typeof(value) != 'undefined')
            {
                for(i in value)
                {
                    //console.log(value[i]);
                    if(value[i].raw.disablefield > 0 )
                    {
                        finalValue.push(value[i]);
                    }
                }
            }else{

                if(value.raw.disablefield > 0 )
                {
                  finalValue =  value;
              }
            }
            value = finalValue;
        }

        this.value = value;
        if(!value)
        {
            if(this.allSelector != false) this.allSelector.removeCls('x-boundlist-selected');
            return this.callParent(arguments);
        }


        if(typeof value == 'string')
        {
            var    me = this,
                records = [],
                vals = value.split(',');

            if(value == '')
            {
                if(me.allSelector != false) me.allSelector.removeCls('x-boundlist-selected');
            }
            else
            {
                if(vals.length == me.store.getCount() && vals.length != 0)
                {
                    if(me.allSelector != false) me.allSelector.addCls('x-boundlist-selected');
                    else me.afterExpandCheck = true;
                }
            }


            Ext.each(vals, function(val)
            {
                var record = me.store.getById(parseInt(val));
                if(record.get('disablefield') > 0)
                if(record) records.push(record);
            });
            return me.setValue(records);
        }
        else return this.callParent(arguments);
    },

    // set value in combo list
    expand: function()
    {
        var    me = this,
            bodyEl, picker, collapseIf;


            if(me.rendered && !me.isExpanded && !me.isDestroyed)
            {
            bodyEl = me.bodyEl;
            picker = me.getPicker();
            collapseIf = me.collapseIf;


            // show the picker and set isExpanded flag
            picker.show();
            me.isExpanded = true;
            me.alignPicker();
            bodyEl.addCls(me.openCls);

            if(me.noData == false) me.noData = picker.getEl().down('.x-boundlist-list-ct').insertHtml('beforeBegin', '<div class="x-boundlist-item" role="option">'+me.noDataText+'</div>', true);
            if(me.addAllSelector == true && me.allSelector == false)
            {
                var selectedvalues =  this.value ;
                  var vals =  String(selectedvalues).split(",");
                if(vals.length == me.store.getCount() && vals.length != 0)
                {
                me.allSelector = picker.getEl().down('.x-boundlist-list-ct').insertHtml('beforeBegin', '<div class="x-boundlist-item x-boundlist-selected" role="option"><span  class="x-combo-checker">&nbsp;</span> '+me.allText+'</div>', true);
                }
                else
                {
                 me.allSelector = picker.getEl().down('.x-boundlist-list-ct').insertHtml('beforeBegin', '<div class="x-boundlist-item" role="option"><span class="x-combo-checker">&nbsp;</span> '+me.allText+'</div>', true);
                }
                me.allSelector.on('click', function(e)
                {
                    if(me.allSelector.hasCls('x-boundlist-selected'))
                    {
                        me.allSelector.removeCls('x-boundlist-selected');
                        me.setValue('');
                        me.fireEvent('select', me, []);
                    }
                    else
                    {
                        var records = [];
                        me.store.each(function(record)
                        {
                            records.push(record);
                        });
                        me.allSelector.addCls('x-boundlist-selected');
                        me.select(records);
                        me.fireEvent('select', me, records);
                    }
                });


                if(me.allSelectorHidden == true) me.allSelector.hide();
                else me.allSelector.show();

                if(me.afterExpandCheck == true)
                {
                    me.allSelector.addCls('x-boundlist-selected');
                    me.afterExpandCheck = false;
                }
            }

            // monitor clicking and mousewheel
            me.mon(Ext.getDoc(),
            {
                mousewheel: collapseIf,
                mousedown: collapseIf,
                scope: me
            });
            Ext.EventManager.onWindowResize(me.alignPicker, me);
            me.fireEvent('expand', me);
            me.onExpand();
        }
        else
        {
            me.fireEvent('expand', me);
            me.onExpand();
        }
    },

    onListSelectionChange: function(list, selectedRecords)
    {
        var    me = this,
            isMulti = me.multiSelect,
            hasRecords = selectedRecords.length > 0;
        // Only react to selection if it is not called from setValue, and if our list is
        // expanded (ignores changes to the selection model triggered elsewhere)
        if(!me.ignoreSelection && me.isExpanded)
        {

            if(!isMulti)
            {
                Ext.defer(me.collapse, 1, me);
            }
            /*
            * Only set the value here if we're in multi selection mode or we have
            * a selection. Otherwise setValue will be called with an empty value
            * which will cause the change event to fire twice.
            */
            if(isMulti || hasRecords)
            {
                me.setValue(selectedRecords, false);
            }
            if(hasRecords)
            {
                me.fireEvent('select', me, selectedRecords);
            }
            me.inputEl.focus();

            if(me.addAllSelector == true && me.allSelector != false)
            {
                if(selectedRecords.length == me.store.getTotalCount()) me.allSelector.addCls('x-boundlist-selected');
                else me.allSelector.removeCls('x-boundlist-selected');
            }
        }
    }
});
```

# Plugin
### xtype : percheckcombo

```javascript
Ext.define('Ext.ux.PerSearchCheckCombo',{
    extend: 'Ext.ux.CheckCombo',
    alias: 'widget.persearchcheckcombo',
    frame: false,
    matchFieldWidth: false,
    searchcombo:false,
    checkComboId:[],
    reloadCombo:false,

    onBeforeDeselect: function(combo, record, index, eOpts)
    {
        var me = this;
        var index = me.checkComboId.indexOf(record);
        if (index > -1)
        {
            me.checkComboId.splice(index, 1);
        }
    },
    onBeforeSelect: function(combo, record, index, eOpts)
    {
        var me = this;
        var index = me.checkComboId.indexOf(record);
        if (index < 0)
        {
            me.checkComboId.push(record);
        }
    },

    createPicker: function()
    {
        var me = this,
        picker,
        menuCls = Ext.baseCSSPrefix + 'menu',
        opts = Ext.apply(
        {
            pickerField: me,
            selModel:
            {
                mode: me.multiSelect ? 'SIMPLE' : 'SINGLE'
            },
            floating: true,
            hidden: true,
            ownerCt: me.ownerCt,
            cls: me.el.up('.' + menuCls) ? menuCls : '',
            store: me.store,
            displayField: me.displayField,
            focusOnToFront: false,
            pageSize: me.pageSize,
            disableFieldFeature: false,
            bind: {
                store: this.store
            },
             listeners: {
                    beforeselect: function(combo, record, index ) {
                        if (record.get(me.disableField) != 0) {
                            // combo.setValue(record.get(me.displayField));
                            // combo.setValue(record.get(me.displayField));
                            // console.log(combo);
                        } else {
                            return false;
                        }
                        // Cancel the default action
                    }

                },

            tpl:
            [
                '<tpl if="'+me.disableFieldFeature+' == '+true+'">',
                    '<tpl><ul><tpl for=".">',

                     //    '<li role="option"   class="' + Ext.baseCSSPrefix + 'boundlist-item"><span class="x-combo-checker">&nbsp;</span> {' + me.displayField + '}</li>',
                        '<tpl if="disablefield != \'0\' ">',
                            '<li role="option"   class="' + Ext.baseCSSPrefix + 'boundlist-item"><span class="x-combo-checker">&nbsp;</span> {' + me.displayField + '}</li>',
                        '<tpl else>',
                            '<li role="option"   class="' + Ext.baseCSSPrefix + 'boundlist-item" ><span><input type="checkbox" disabled="disabled" />&nbsp;</span> {' + me.displayField + '}</li>',
                        '</tpl>',
                    '</tpl></ul></tpl>' ,
                '<tpl else>',
                    '<tpl><ul><tpl for=".">',
                        '<li role="option"  class="' + Ext.baseCSSPrefix + 'boundlist-item"><span class="x-combo-checker">&nbsp;</span> {' + me.displayField  + '}</li>',
                    '</tpl></ul></tpl>' ,
                '</tpl>'
            ]
        }, me.listConfig, me.defaultListConfig);

        picker = me.picker = Ext.create('Ext.view.BoundList', opts);
        if(me.pageSize)
        {
            picker.pagingToolbar.on('beforechange', me.onPageChange, me);
        }

        me.mon(picker,
        {
            itemclick: me.onItemClick,
            refresh: me.onListRefresh,
            scope: me
        });

        me.mon(picker.getSelectionModel(),
        {
            'beforeselect': me.onBeforeSelect,
            'beforedeselect': me.onBeforeDeselect,
            'selectionchange': me.onListSelectionChange,
            scope: me
        });

        me.store.on('load', function(store)
        {
            if(store.getTotalCount() == 0)
            {
                me.allSelectorHidden = true;
                if(me.allSelector != false) me.allSelector.setStyle('display', 'none');
                if(me.noData != false) me.noData.setStyle('display', 'block');
            }
            else
            {
                me.allSelectorHidden = false;
                if(me.allSelector != false) me.allSelector.setStyle('display', 'block');
                if(me.noData != false) me.noData.setStyle('display', 'none');
            }
        });
        me.checkComboId = [];
        return picker;
    },

    setValue: function(value)
    {
         if(typeof value != 'undefined' && this.disableFieldFeature === true)
        {
                var finalValue = [];
                if(typeof(value) != 'undefined')
                {
                    for(i in value)
                    {
                       // console.log(value[i]);
                        if(value[i].raw.disablefield > 0 )
                        {
                            finalValue.push(value[i]);
                        }
                    }
                }else{
                  //  console.log('in else');
                    if(value.raw.disablefield > 0 )
                    {
                      finalValue =  value;
                  }
                }
                value = finalValue;
            }

        this.value = value;
        if(!value)
        {
            if(this.allSelector != false) this.allSelector.removeCls('x-boundlist-selected');
            return this.callParent(arguments);
        }

        if(typeof value == 'string')
        {
            var    me = this,
                records = [],
                vals = value.split(',');

            if(value == '')
            {
                if(me.allSelector != false) me.allSelector.removeCls('x-boundlist-selected');
            }
            else
            {
                if(vals.length == me.store.getCount() && vals.length != 0)
                {
                    if(me.allSelector != false) me.allSelector.addCls('x-boundlist-selected');
                    else me.afterExpandCheck = true;
                }
            }

            Ext.each(vals, function(val)
            {
                var record = me.store.getById(parseInt(val));
                if(record.get('disablefield') > 0)
                if(record) records.push(record);
            });


            return me.setValue(records);
        }
        else return this.callParent(arguments);
    },

    getValue: function()
    {
                if(this.value != null)
        if(typeof this.value == 'object') {return this.value.join(',');}
        else { return this.value;}
    },

    // it will set value in combo list
    expand: function()
    {
        var    me = this,
        bodyEl, picker, collapseIf;

        if(me.rendered && !me.isExpanded && !me.isDestroyed)
        {
            bodyEl = me.bodyEl;
            picker = me.getPicker();
            collapseIf = me.collapseIf;

            // show the picker and set isExpanded flag
            picker.show();
            me.isExpanded = true;
            me.alignPicker();
            bodyEl.addCls(me.openCls);

            var searchfld = null;
            var searchfld = "searchfield_"+me.id;
            if((this.value=='') || (this.value==null))
            {
                this.checkComboId = [];
            }

            if(me.searchcombo == false)
            {
                if(document.getElementById(searchfld))
                {
                    document.getElementById(searchfld).remove();
                }
                me.searchField = picker.getEl().down('.x-boundlist-list-ct').insertHtml('beforeBegin','<div class="x-boundlist-item" style="padding-top:5px;padding-bottom:0px;"><input autofocus="autofocus" style="width:90%" type="textfield" name="'+searchfld+'" id="'+searchfld+'" placeholder="Enter Search Text..."></div>', true);
                me.searchcombo = true;
            }
            me.fireEvent('blur', me, []);
            document.getElementById(searchfld).focus();
            document.getElementById(searchfld).value = '';
            document.getElementById(searchfld).focus();
            me.store.clearFilter();

            var original_store_count = me.store.getCount();

            if(me.addAllSelector == true && me.allSelector != false){
                if(original_store_count > 0){
                    me.allSelector.setStyle('display', 'block');
                }
            }

            me.searchField.on('keyup', function(e)
            {
                var searchVal = document.getElementById(searchfld).value;

                //console.log(searchVal);
                //console.log(me.store);

                if((searchVal == '') || (searchVal  == null))
                {
                    me.store.clearFilter();
                    if(me.store.getCount() == 0)
                    {
                        me.noData.setStyle('display', 'block');
                    }
                    else
                    {
                        me.noData.setStyle('display', 'none');
                    }
                    if(me.addAllSelector == true && me.allSelector != false)
                    {
                        me.allSelector.setStyle('display', 'block');

                        if((me.checkComboId.length == original_store_count) && (me.checkComboId.length!=0))
                        {
                            me.allSelector.addCls('x-boundlist-selected');
                        }
                        else
                        {
                            me.allSelector.removeCls('x-boundlist-selected');
                        }
                    }
                }
                me.store.clearFilter();

                me.store.filter([
                        {
                        property     : me.displayField,
                        value        : searchVal,
                        anyMatch     : true, //optional, defaults to true
                        caseSensitive: false//optional, defaults to true
                        }
                ]);
                var filter_store_count = me.store.getCount();
                if((searchVal != '') && (searchVal  != null) && (filter_store_count > 0))
                {
                    if(filter_store_count!=original_store_count)
                    {
                        if(me.allSelector != false) me.allSelector.setStyle('display', 'none');
                    }
                    else
                    {
                        if(me.allSelector != false) me.allSelector.setStyle('display', 'block');
                    }
                    if(filter_store_count == 0)
                    {
                        me.noData.setStyle('display', 'block');
                    }
                    else
                    {    me.noData.setStyle('display', 'none');
                    }
                }
                document.getElementById(searchfld).focus();
            });

            if(me.noData == false) me.noData = picker.getEl().down('.x-boundlist-list-ct').insertHtml('beforeBegin', '<div class="x-boundlist-item" role="option">'+me.noDataText+'</div>', true);

            if(original_store_count == 0){
                me.noData.setStyle('display', 'block');
            }else{
                me.noData.setStyle('display', 'none');
            }

            if(me.addAllSelector == true && me.allSelector == false)
            {
                var selectedvalues =  this.value ;

                if((selectedvalues!='') && (selectedvalues!=null)){
                        var vals =  String(selectedvalues).split(",");
                }else{
                        var vals = [];
                }


                if(vals.length == me.store.getCount() && vals.length != 0)
                {
                    me.allSelector = picker.getEl().down('.x-boundlist-list-ct').insertHtml('beforeBegin', '<div class="x-boundlist-item x-boundlist-selected" role="option"><span  class="x-combo-checker">&nbsp;</span> '+me.allText+'</div>', true);
                }
                else
                {
                    me.allSelector = picker.getEl().down('.x-boundlist-list-ct').insertHtml('beforeBegin', '<div class="x-boundlist-item" role="option"><span class="x-combo-checker">&nbsp;</span> '+me.allText+'</div>', true);
                }

                me.allSelector.on('click', function(e)
                {
                    if(me.allSelector.hasCls('x-boundlist-selected'))
                    {
                        me.allSelector.removeCls('x-boundlist-selected');
                        me.setValue('');
                        me.fireEvent('select', me, []);
                    }
                    else
                    {
                        var records = [];
                        me.store.each(function(record)
                        {
                            records.push(record);
                        });
                        me.allSelector.addCls('x-boundlist-selected');
                        me.select(records);
                        me.fireEvent('select', me, records);
                    }
                });

                if(me.allSelectorHidden == true) me.allSelector.hide();
                else me.allSelector.show();

                if(me.afterExpandCheck == true)
                {
                    me.allSelector.addCls('x-boundlist-selected');
                    me.afterExpandCheck = false;
                }
            }

            // monitor clicking and mousewheel
            me.mon(Ext.getDoc(),
            {
                mousewheel: collapseIf,
                mousedown: collapseIf,
                scope: me
            });
            Ext.EventManager.onWindowResize(me.alignPicker, me);
            me.fireEvent('expand', me);
            me.onExpand();
        }
        else
        {
            me.fireEvent('expand', me);
            me.onExpand();
        }
    },

    alignPicker: function()
    {
        var me = this,
        picker = me.getPicker();
        me.callParent();
        if(me.addAllSelector == true)
        {
            var height = picker.getHeight();
            if(picker.getEl().select('.x-boundlist-item').elements.length > 15){
                height = parseInt(height)+25;
                picker.setHeight(height);
                picker.getEl().setStyle('height', height+'px');
            }
        }
        if(me.searchcombo == true)
        {
            var height = picker.getHeight();
            if(picker.getEl().select('.x-boundlist-item').elements.length > 15){
                if(me.addAllSelector == true){
                    height = parseInt(height)+15;
                }else{
                    height = parseInt(height)+25;
                }
                picker.setHeight(height);
                picker.getEl().setStyle('height', height+'px');
            }
        }
    },

    onListSelectionChange: function(list, selectedRecords)
    {
        var    me = this,
            isMulti = me.multiSelect,
            hasRecords = selectedRecords.length > 0;
        // Only react to selection if it is not called from setValue, and if our list is
        // expanded (ignores changes to the selection model triggered elsewhere)
        if(!me.ignoreSelection && me.isExpanded)
        {

            if(!isMulti)
            {
                Ext.defer(me.collapse, 1, me);
            }
            /*
            * Only set the value here if we're in multi selection mode or we have
            * a selection. Otherwise setValue will be called with an empty value
            * which will cause the change event to fire twice.
            */
            if(isMulti || hasRecords)
            {
                me.setValue(selectedRecords, false);
            }
            if(hasRecords)
            {
                me.fireEvent('select', me, selectedRecords);
            }
            me.inputEl.focus();

            if(me.addAllSelector == true && me.allSelector != false)
            {
                //if(selectedRecords.length == me.store.getTotalCount()) me.allSelector.addCls('x-boundlist-selected');
                if(me.checkComboId.length == me.store.getTotalCount()) me.allSelector.addCls('x-boundlist-selected');
                else me.allSelector.removeCls('x-boundlist-selected');
            }
        }
    }
});
```
Happy Coding! ☺