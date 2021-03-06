<template>
    <table class="table">
        <caption>
            <h3>Field</h3>
        </caption>
        <thead>
            <tr>
                <th width="130px"></th>
                <th>Name</th>
                <th>Type</th>
                <th>Length / Enum</th>
                <th>Default / Comment / Allow Null</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="field in manager.list" :key="field.name">
                <td class="button-cell">
                    <div class="btn-group">
                        <span v-on:click="remove(field)" class="btn btn-danger"> X </span>
                        <span v-on:click="manager.moveUp(field)" class="btn btn-default"> ↑ </span>
                        <span v-on:click="manager.moveDown(field)" class="btn btn-default"> ↓ </span>
                    </div>
                </td>
                <td>
                    <span v-on:click="rename(field)" class="btn btn-default">{{ field.name }}</span>
                </td>
                <td>
                    <span v-on:click="setType(field)" class="btn btn-default">{{ field.type }}</span>
                </td>
                <td><input v-if="field.hasLength" v-model="field.length" class="form-control" type="text" /></td>
                <td>
                    <FieldPanel :field="field"></FieldPanel>
                </td>
            </tr>
        </tbody>
        <tfoot>
            <tr>
                <td>
                    <select v-model="selected" @change="addType($event.target.value)" class="form-control">
                        <option selected="true" disabled="disabled" value="">...</option>
                        <option v-for="name in CommonTypeList" :key="name">{{ name }}</option>
                    </select>
                </td>
                <td>
                    <select @change="addField($event.target.value, IntegerFieldList)" class="form-control">
                        <option selected="true" disabled="disabled">...</option>
                        <option v-for="field in IntegerFieldList" :value="field.name" :key="field.name">{{ field.name }}</option>
                    </select>
                </td>
                <td>
                    <select @change="addField($event.target.value, CommonFieldList)" class="form-control">
                        <option selected="true" disabled="disabled">...</option>
                        <option v-for="field in CommonFieldList" :value="field.name" :key="field.name">{{ field.name }}</option>
                    </select>
                </td>
                <td>
                    <select @change="addName($event.target.value)" class="form-control">
                        <option selected="true" disabled="disabled">...</option>
                        <option v-for="entity in EntityList" :key="entity.name">{{ entity.tableName }}_id</option>
                    </select>
                </td>
                <td>
                    <span v-on:click="add" class="btn btn-primary plus"> + </span>
                </td>
            </tr>
        </tfoot>
    </table>
</template>

<script>
    import bus from '../helper/event';
    import { see, sure, enter } from '../helper/dialogue';
    import { CommonFieldList, CommonTypeList, FieldTypeList, IntegerFieldList } from '../helper/field';
    import FieldPanel from './FieldPanel';
    import { LDData } from './ListDialogue';

    export default {
        name: 'Field',
        props: ['manager'],
        components: { FieldPanel },
        data() {
            return {
                EntityList: bus.project.EntityManager.list,
                CommonFieldList,
                CommonTypeList,
                IntegerFieldList,
                selected: ''
            };
        },
        methods: {
            add() {
                try {
                    LDData.show('Select a Type', FieldTypeList, 'type', null, field => {
                        const fff = this.manager.cloneType(field.type);
                        this.manager.add(fff);
                        this.rename(fff);
                    });
                } catch (error) {
                    see(error, 400);
                }
            },
            addField(name, list) {
                try {
                    const found = list.find(item => item.name === name);
                    const fff = this.manager.make(found.name, found.type);
                    fff.load(found);
                    this.manager.add(fff);
                } catch (error) {
                    see(error, 400);
                }
            },
            addName(name) {
                try {
                    const fff = this.manager.make(name, 'integer');
                    this.manager.add(fff);
                } catch (error) {
                    see(error, 400);
                }
            },
            addType(type) {
                try {
                    this.selected = '';
                    const field = this.manager.cloneType(type);
                    this.manager.add(field);
                    this.rename(field);
                } catch (error) {
                    see(error, 400);
                }
            },
            remove(field) {
                sure('Are you sure?').then(result => {
                    if (result.value) {
                        this.manager.remove(field);
                    }
                });
            },
            rename(field) {
                enter('Please enter the Field name', field.name).then(result => {
                    if (result.value) {
                        try {
                            field.name = result.value;
                        } catch (error) {
                            see(error, 400);
                        }
                    }
                });
            },
            setType(field) {
                LDData.show('Select a Type', FieldTypeList, 'type', null, result => {
                    field.type = result.type;
                });
            }
        }
    };
</script>
