# 基本

- layout: default
- order: 0

最简单的用法。

---

## 可执行 DEMO

````js
import Form, { FormItem, FormCore, If, Item } from 'noform';
import repeater  from '../src/repeater';
import * as Antd from 'antd';
import wrapper from '../src/wrapper/antd';
import dialogWrapper from '../src/dialog/antd';
import "./repeater.scss";

const { Table, Button, Input, Radio, Checkbox, Select, InputNumber }  = wrapper(Antd);
const Dialog = dialogWrapper(Antd)
const { TableRepeater, InlineRepeater, Selectify, ActionButton } = repeater({ Dialog, Button, Input, Checkbox, Radio });

const validateConfig = {
    username: { type: 'string', required: true },
};

const publicCountry = [
    { label: 'US', value: 'US' },
    { label: 'GB', value: 'GB' },
    { label: 'AU', value: 'AU' },
    { label: 'CN', value: 'CN' },
    { label: 'BR', value: 'BR' },
    { label: 'EG', value: 'EG' },
    { label: 'DE', value: 'DE' },
];
class Example extends React.Component {
    constructor(props, context) {
        super(props, context);
        this.core = new FormCore({
            values: {
                countryRepeater: [
                    { country: ['US'] },
                    { country: ['AU', 'GB'] }
                ],
                ruleRepeater: [
                    { min: 5, max: 10 },
                    { min: 24, max: 39 },
                    { min: 45, max: 60 },
                ],
                masterRepeater: [
                    // { check: 'bbb'},
                    // { check: 'aaa'}
                    { username: 0 },
                    { username: 0.0 },
                    { username: 1 }
                ],
                inlineRepeater: [
                    { username: 'username1', s1: 's1', s2: 's2', s3: 's3', source: 1, },
                    // { username: 'username2',s1: 's1...', s2: 's2...', s3: 's3...', source: 2 }
                ],
                inlineRepeater_a: [
                    // { hidValidate: null, source: false, dep: false, list: [ { username: 'tom', sublist: [{ subname: '111' }] }] }
                    // { triggerDiscount: false, discount: 100 },
                    {
                        source2: '2',
                        source3: '3',
                        target: 'helloworld',
                        triggerIsNormal: false,
                        isNormal: true,
                    }
                ],
                products: [
                    { name: 123}
                ],
                int1: 0,
                int2: 1
            },
            autoValidate: true,
            validateConfig: {                
                inlineRepeater: { type: 'array', required: true, messgae: 'hull' }
            },
            onChange: (fireKeys, values, ctx) => {
                const { currentEventOpts } = ctx;
                const { type } = currentEventOpts || {};
                setTimeout(() => {
                    const withRender = type !== 'add';
                    ctx.validateItem('inlineRepeater', undefined, { withRender });
                }, 500);
            }
        });

        window.core = this.core;
        this.formConfig = {
            validateConfig,
            autoValidate: true,
        };
        
        this.publicHandler = {
            afterSetting: (event, repeater) => {
                console.log('event', event, 'repeater', repeater);
                const { formList } = repeater;
                const options = this.getRestCountry(formList);
                formList.forEach((fc) => {
                    fc.setItemProps('country', { options })
                });
                this.core.setItemValue('restCountryDEMO', options);
                this.core.setItemProps('countryRepeater', { hasAdd: (options.length !== 0) });
            }
        };

        this.ruleHandler = {
            afterSetting: (event, repeater) => {
                const { type, index: modifyIndex } = event;
                const { formList } = repeater;
                const values = repeater.getValues();

                console.log('event', event, repeater);

                formList.forEach((ctx, index) => {
                    const validateConfig = this.getValidateConfig(values, index);
                    ctx.setValidateConfig(validateConfig);
                    if (ctx.timer) clearTimeout(ctx.timer);
                    ctx.timer = setTimeout(() => {
                        ctx.validate();    
                    }, 300);
                });
            }
        };
    }

    getValidateConfig = (values, index) => {
        const empty = { type: 'number' };
        const before = values[index - 1];
        const after = values[index + 1];
        const current = values[index];

        const minMin = before && !isNaN(before.max) ? Number(before.max) : null;
        const minMax = current && !isNaN(current.max) ? Number(current.max) : null;

        const maxMin = current && !isNaN(current.min) ? Number(current.min) : null;
        const maxMax = after && !isNaN(after.min) ? Number(after.min) : null;

        console.log(`[${index}]`, [minMin, minMax], [maxMin, maxMax]);

        return {
            min: [
                minMin ? { type: 'number', min: minMin, message: `[${index}]不能小于${minMin}` } : empty,
                minMax ? { type: 'number', max: minMax, message: `[${index}]不能大于${minMax}` } : empty
            ],
            max: [
                maxMin ? { type: 'number', min: maxMin, message: `[${index}]不能小于${maxMin}` } : empty,
                maxMax ? { type: 'number', max: maxMax, message: `[${index}]不能大于${maxMax}` } : empty
            ]
        }
    }

    getRestCountry = (formList) => {
        let restCountry = [...publicCountry];
        formList.forEach(fc => { // 公共池去除本次已选取的
            const countryCode = fc.getItemValue('country') || [];
            restCountry = restCountry.filter(r => countryCode.indexOf(r.value) === -1);
        });

        return restCountry;
    }

    renderTableView = (_, ctx) => {
        const { setLoading, getLoading, getDataSource, getCoreList } = ctx;

        const coreList = getCoreList();
        const dataSource = getDataSource() || [];
        const loading = getLoading();        

        const elements = dataSource.map((item, index) => {            
            const core = coreList[index];
            console.log('coreList', core);

            return <div>
                <div>loading: {loading ? 'on' : 'off'}</div>
                <div>username: {item.username}</div>
                <Button onClick={setLoading.bind(null, true)}>Turn ON Loading</Button>
                <Button onClick={setLoading.bind(null, false)}>Turn OFF Loading</Button>
                <ActionButton core={core} type="delete">删除{index}</ActionButton>
            </div>
        });    

        return <div>
            {elements}
        </div>
    }

    renderTest = () => {
        const repeaterDialogConfig = {
            layout: { label: 6, control: 16 },
            title: 'title123',
            // defaultMinWidth: false,
            // style: { width: '800px' },
            // hasCancel: false,
            // btnAlign: 'left',
        };
        const inlineWrap = { style: { display: 'flex' }};
        const inlinePrefix = { layout: null, inline: true };
        const inlineSuffix = { layout: { label: 6, control: 18 }, inline: true, full: true, style: { flex: 1 }};
        return <FormItem label="masterRepeater" name="masterRepeater" full>
            <TableRepeater
                // view={this.renderTableView}
                dialogConfig={repeaterDialogConfig}
            >
                <div>hello1</div>
                <FormItem name="username" label="username"><Input /></FormItem>
            </TableRepeater>
        </FormItem>
    }

    handleView = (_, ctx) => {
        const dataSource = ctx.getDataSource();
        const formList = ctx.getCoreList();

        return formList.map((core) => {
            return <Form core={core}>
                <FormItem label="sx" name="sx">
                    <Input />
                </FormItem>
                <FormItem label="s1" name="s1" status="preview">
                    <Input />
                </FormItem>
                <FormItem label="s2" name="s2" status="disabled">
                    <Input />
                </FormItem>
                <FormItem label="s3" name="s3" status={() => {
                    return 'preview'
                }}>
                    <Input />
                </FormItem>
                <If when={(values) => {
                    return !values.noPrice;
                }} >
                    <FormItem label="pp" name="pp">
                        <Input />
                    </FormItem>
                </If>
            </Form>
        });
    }

    renderView = (_, ctx) => {
        const dataSource = ctx.getDataSource(); // 获取Repeater当前数据源
        const coreList = ctx.getCoreList(); // 获取Repeater form核心列表
        const isLoading = ctx.getLoading(); // 获取loading状态, 通过setLoading更改

        const cellProps = (rowIndex, colIndex) => {
            console.log('===>', rowIndex, colIndex);
            if (rowIndex === 0 && colIndex === 0) {
                return {
                    colSpan: 1,
                    rowSpan: dataSource.length
                };
            }
        };

        return <Table dataSource={dataSource} cellProps={cellProps}>
            <Table.Column title="username" dataIndex="username"/>
            <Table.Column title="source" render={(_, record, index) => {
                window.xc = coreList[index];
                return <FormItem name="source" core={coreList[index]}>
                    <Input />
                </FormItem>
            }} />
            <Table.Column title="conditional" render={(_, record, index) => {
                return <If core={coreList[index]} when={(v) => {
                    return v.source === 'a';
                }}>
                    <FormItem name="cond" core={coreList[index]}>
                        <Input />
                    </FormItem>
                </If>
            }} />
            <Table.Column title="validate" render={(_, record, index) => {
                return <div>
                    <Button onClick={() => {
                        coreList[index].validate();
                    }}>validate</Button>
                    <ActionButton core={coreList[index]} type="addMultipleInline" render={(currentCore, rptCore) => {
                        const len = rptCore.formList.length;
                        return <div>当前长度：{len}</div>
                    }}/>
                </div>
            }} />
        </Table>
        
    }

    render() {
        const fields = [
            'a',
            'b',
        ];

        const typeSource = [
            { label: 'aaa', value: 'aaa' },
            { label: 'bbb', value: 'bbb' },
            { label: 'ccc', value: 'ccc' }
        ];

        const asyncHandlerInline = {
            add: () => {
            }
        };

        return (<Form core={this.core} layout={{ label: 6, control: 18 }} defaultMinWidth={false}>
            <div className="example-title">Master Repeater Examples</div>
            {/* {this.renderTest()} */}
            {/* <FormItem label="int1" name="int1" status="preview" defaultValue="2">
                <Input />
            </FormItem>

            <FormItem label="int2" name="int2" status="preview">
                <Input />
            </FormItem> */}
            {/* <FormItem label="dynamic repeater" name="inlineRepeater">
                <InlineRepeater multiple maxLength={2} addText={(currentCore, rptCore) => {
                    const len = rptCore.formList.length;
                        return <div>当前长度l：{len}</div>
                }}>
                    <FormItem label="s1" name="s1" status="preview">
                        <Input />
                    </FormItem>
                    <FormItem label="s2" name="s2" status="disabled">
                        <Input />
                    </FormItem>
                    <FormItem label="s3" name="s3" status={() => {
                        return 'preview'
                    }}>
                        <Input />
                    </FormItem>
                </InlineRepeater>
            </FormItem> */}
            {/* <FormItem label="dynamic repeater" name="inlineRepeater">
                <InlineRepeater multiple view={this.handleView} />
            </FormItem> */}
            {/* <FormItem label="hide price" name="noPrice"><Checkbox /></FormItem>
            <Item render={(values) => {
                const { noPrice = false } = values;

                return <FormItem name="products">
                    <TableRepeater>
                        <FormItem label="name" name="name"><Input /></FormItem>
                        { noPrice ? null : <FormItem label="price" name="price"><Input /></FormItem> }
                    </TableRepeater>
                </FormItem>
            }} /> */}

            <FormItem label="dynamic repeater" name={`inlineRepeater_a`}>
                <InlineRepeater
                    // hasAdd={false}
                    hasAdd
                    multiple
                    formConfig={{
                        validateConfig: {
                            source: { required: true, message: 'source is required' },
                            hidValidate: { required: true, message: 'hidValidate is required' },
                        },
                        autoValidate: true,
                        onChange: (fireKeys, currentValues, ctx) => {
                            if (fireKeys.length === 1) {
                                const [fk] = fireKeys;
                                if (['source1', 'source2', 'source3'].includes(fk)) {
                                    ctx.repeater.formList.forEach(f => {
                                        f.setItemValue('target', '');
                                    });
                                }
                            }
                        },
                        interceptor: {
                            source1: (newValue, oldValue, ctx) => {
                                if (oldValue !== undefined &&
                                    newValue !== oldValue) {
                                    return new Promise((resolve) => {
                                        const ins = Dialog.show({
                                            title: '提示',
                                            content: <div>确定更改</div>,
                                            onOk: () => {
                                                resolve(newValue);
                                                ins.hide();
                                            },
                                            cancel: () => {
                                                resolve(oldValue);
                                                ins.hide();
                                            }
                                        });
                                    });
                                } else {
                                    return newValue;
                                }
                            }
                        },
                    }}
                    asyncHandler={{
                        afterSetting: (event, repeater) => {
                            const { changeKeys, type, index, core } = event;
                            if (type === 'update' && changeKeys.length === 1 && changeKeys.includes('target') || 
                                type === 'delete') {
                                
                                const values = repeater.getValues();
                                const sortedValues = [...values];

                                if (type === 'update') {
                                    const { target } = core.getValues();
                                    if (target === 'abc') {
                                        sortedValues.push({ target: 'def', isNormal: true, });
                                    }
                                }
                                

                                window.core.setItemValue('inlineRepeater_a', sortedValues);
                            }
                        }
                    }}
                    view={(_, ctx) => {
                        const dataSource = ctx.getDataSource(); // 获取Repeater当前数据源
                        const coreList = ctx.getCoreList(); // 获取Repeater form核心列表

                        return dataSource.map((item, index) => {                            
                            return <div>
                                <FormItem label="target" name="target" core={coreList[index]}>
                                    <Input />
                                </FormItem>
                                { index === 0 ? <FormItem label="other" name="other" core={coreList[index]}>
                                    <Input />
                                </FormItem> : null }
                                <ActionButton core={coreList[index]} type="delete">删除111</ActionButton>
                            </div>
                        });

                        return <Table dataSource={dataSource}>
                            <Table.Column title="source" render={(_, record, index) => {
                                return <div>  
                                    <FormItem name="source1" core={coreList[index]}>
                                        <Radio.Group options={[
                                                { label: 'false', value: false },
                                                { label: 'true', value: true }
                                        ]} />
                                    </FormItem>
                                    <If core={coreList[index]} when={values => values.source1 === true}>
                                        <div>
                                            <FormItem name="followSource1" core={coreList[index]}>
                                                <Input />
                                            </FormItem>
                                        </div>
                                    </If>
                                    <FormItem name="source2" core={coreList[index]}>
                                        <Input />
                                    </FormItem>
                                    <FormItem name="source3" core={coreList[index]}>
                                        <Input />
                                    </FormItem>
                                </div>;
                            }} />
                            <Table.Column title="target" render={(_, record, index) => {
                                return <div>  
                                    <FormItem name="target" core={coreList[index]}>
                                        <Input />
                                    </FormItem>
                                </div>;
                            }} />
                            <Table.Column title="triggerIsNormal" render={(_, record, index) => {
                                return <div>  
                                        <FormItem name="triggerIsNormal" core={coreList[index]}>
                                            <Radio.Group options={[
                                                { label: 'false', value: false },
                                                { label: 'true', value: true }
                                            ]} />
                                        </FormItem>
                                </div>;
                            }} />
                            <Table.Column title="isNormal" render={(_, record, index) => {
                                return <If core={coreList[index]} when={values => values.triggerIsNormal === true}>
                                    <div>
                                        <FormItem name="isNormal" core={coreList[index]}>
                                            <Radio.Group options={[
                                                { label: 'false', value: false },
                                                { label: 'true', value: true }
                                            ]} />
                                        </FormItem>
                                    </div>
                                </If>;
                            }} />
                            <Table.Column title="deepUsage" render={(_, record, index) => {
                                return <If core={coreList[index]} when={values => values.isNormal === true}>
                                    <div>
                                        <FormItem name="deepUsage" core={coreList[index]}>
                                            <Input />
                                        </FormItem>
                                    </div>
                                </If>;
                            }} />
                            {/* <Table.Column title="if2" render={(_, record, index) => {
                                return <If core={coreList[index]} when={values => values.dep === false}>
                                    <FormItem name="hidValidate" core={coreList[index]}>
                                        <Input />
                                    </FormItem>
                                </If>;
                            }} /> */}
                            {/* <Table.Column title="list" render={(_, record, parentIndex) => {
                                return <FormItem errorRender={(err, errInfo) => {
                                    const {main, sub} = errInfo || {};
                                    return sub ? <div style={{ color: 'red' }}>{sub}</div> : null;
                                }} name="list" core={coreList[parentIndex]}>
                                    <InlineRepeater
                                        onMount={(list) => {
                                            window.list = list;
                                        }}
                                        asyncHandler={{
                                            afterSetting: () => {

                                            }
                                        }}
                                        hasAdd={false}
                                        multiple
                                        formConfig={{
                                            validateConfig: {
                                                username: { required: true, message: 'username is required' },
                                                // username: {
                                                //     validator(rule, value, callback, source, options) {
                                                //         if(value !== 'b'){
                                                //             callback(['error']);
                                                //         }
                                                //         callback([])
                                                //     }
                                                // },
                                            },
                                            autoValidate: true,
                                        }}
                                        view={(_, ctx) => {
                                            const dataSource = ctx.getDataSource(); // 获取Repeater当前数据源
                                            const subCoreList = ctx.getCoreList(); // 获取Repeater form核心列表

                                            return <Table dataSource={dataSource} >
                                                <Table.Column title="username" render={(_, record, index) => {
                                                    return <FormItem name="username" core={subCoreList[index]}>
                                                        <Input />
                                                    </FormItem>;
                                                }} />
                                                <Table.Column title="condition" render={(_, record, index) => {
                                                    return <If core={subCoreList[index]} when={(values) => values.username === 'a'}>
                                                        <FormItem name="cond" core={subCoreList[index]}>
                                                            <Input />
                                                        </FormItem>
                                                    </If>;
                                                }} />
                                                <Table.Column title="operation" render={(_, record, index) => {
                                                    console.log('外部', coreList, parentIndex, coreList[parentIndex]);
                                                    return <div>
                                                        <div>外部</div>
                                                        <ActionButton core={coreList[parentIndex]} type="addMultipleInline">新增111</ActionButton>
                                                        <ActionButton core={coreList[parentIndex]} type="delete">删除111</ActionButton>

                                                        <div>本行</div>
                                                        <ActionButton core={subCoreList[index]} type="addMultipleInline">新增</ActionButton>
                                                        <ActionButton core={subCoreList[index]} type="delete">删除</ActionButton>
                                                    </div>
                                                }} />
                                                <Table.Column title="sublist" render={(_, record, parentIndex) => {
                                                    return <FormItem name="sublist" core={subCoreList[parentIndex]}>
                                                        <InlineRepeater
                                                            hasAdd={false}
                                                            multiple
                                                            formConfig={{
                                                                validateConfig: {
                                                                    subname: { required: true, message: 'subname is required' },
                                                                },
                                                                autoValidate: true,
                                                            }}
                                                            view={(_, ctx) => {
                                                                const dataSource = ctx.getDataSource(); // 获取Repeater当前数据源
                                                                const deepCoreList = ctx.getCoreList(); // 获取Repeater form核心列表

                                                                return <Table dataSource={dataSource} >
                                                                    <Table.Column title="subname" render={(_, record, index) => {
                                                                        return <FormItem name="subname" core={deepCoreList[index]}>
                                                                            <Select options={[{ label: 'a', value: 'a' }]} />
                                                                        </FormItem>;
                                                                    }} />
                                                                    <Table.Column title="operation" render={(_, record, index) => {
                                                                        return <div>                                                                            
                                                                            <div>本行</div>
                                                                            <ActionButton core={deepCoreList[index]} type="addMultipleInline">新增</ActionButton>
                                                                            <ActionButton core={deepCoreList[index]} type="delete">删除</ActionButton>
                                                                        </div>
                                                                    }} />
                                                                </Table>
                                                            }}
                                                        />
                                                    </FormItem>
                                                }}/>
                                            </Table>
                                        }}
                                    />
                                </FormItem>
                            }}/> */}
                        </Table>
                    }}
                />
            </FormItem>
            {/* public source */}
            {/* <FormItem label="Public Country Repeater" name="countryRepeater">
                <InlineRepeater asyncHandler={this.publicHandler} multiple>
                    <FormItem label="country" name="country">
                        <Select mode="multiple" options={publicCountry} />
                    </FormItem>
                    <FormItem label="min" name="min"><InputNumber /></FormItem>
                    <FormItem label="abb" name="abb"><InputNumber /></FormItem>
                </InlineRepeater>
            </FormItem> */}
            {/* <FormItem label="Freight Calculator" name="freightCalculator">
                <InlineRepeater multiple>
                    <FormItem label="type" name="type" defaultValue="aaa">
                        <Select options={typeSource} />
                    </FormItem>
                    <FormItem label="rule" multiple>
                        <div>
                            <If when={(values) => values.type === 'aaa'}>
                                <Item name="aaa"><Input placeholder="aaa" /></Item>
                            </If>
                            <If when={(values) => values.type === 'bbb'}>
                                <Item name="bbb"><Input placeholder="bbb" /></Item>
                            </If>
                            <If when={(values) => values.type === 'ccc'}>
                                <Item name="ccc"><Input placeholder="ccc" /></Item>
                            </If>
                        </div>
                    </FormItem>
                </InlineRepeater>
            </FormItem> */}
            {/* <FormItem render={(values) => {
                const availableCountry = (values.restCountryDEMO || []).map(item => item.label).join(', ');
                return <div>Available Country: <span style={{ color: 'red' }}>{availableCountry}</span></div>
            }} />
            
            <FormItem label="Rule Repeater" name="ruleRepeater">
                <InlineRepeater asyncHandler={this.ruleHandler} multiple formConfig={{ autoValidate: true }}>
                    <FormItem label="min" name="min"><InputNumber /></FormItem>
                    <FormItem label="max" name="max"><InputNumber /></FormItem>
                </InlineRepeater>
            </FormItem> */}
        </Form>);
    }
}

ReactDOM.render(<Example />, mountNode);

````

````css
body {
    /* background-color: #000; */
    /* background: #000 !important; */
    /* color: #fff !important; */
}
````