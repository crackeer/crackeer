---
title: "Nextjs"
date: 2021-01-31T09:42:08+08:00
tags: ["nextjs"]

---

#### 一个form的组件编写

```javascript
import React from 'react';
import { Form, Input, Button, Checkbox, message } from 'antd';
import APIOpen from '../../api/open';
import { Select, Radio, Divider } from 'antd';
import JSONEditor from '../../component/JSONEditor';
import { PlusOutlined } from '@ant-design/icons';
import { threadId } from 'worker_threads';
const { Option } = Select;
const { TextArea } = Input;
const layout = {
    labelCol: { span: 4 },
    wrapperCol: { span: 16 },
};
const tailLayout = {
    wrapperCol: { offset: 11, span: 2 },
};

class FormX extends React.Component<any, any> {
    form: any = null;
    belongList: any = null;
    serviceList: any = null;

    json1: any = null;
    json2: any = null;
    json3: any = null;
    constructor(props: any) {
        super(props); // 用于父子组件传值
        this.state = {
            apiList: [],

            service: '',
            api: '',

            extension: {},
            config: [[]],
            response: {},
            addBelong: '',

            configJSON: true,
        }
        this.getBelongList()
        this.getServiceList()
    }
    async getBelongList() {
        let res = await APIOpen.distinct('router', 'belong', {})
        let l = []
        res.data.forEach(item => {
            if (item.belong != null && item.belong.length > 0) {
                l.push({
                    label: item.belong,
                    value: item.belong
                })
            }
        })
        this.belongList = l
    }
    async getServiceList() {
        let res = await APIOpen.getList('service', {
            env: 'default'
        })
        let l = []
        res.data.forEach(item => {
            l.push({
                label: item.name + '【' + item.namespace + '】',
                value: item.namespace
            })
        })
        this.serviceList = l
    }
    changeService = async (value) => {
        let res = await APIOpen.getList('service_api', {
            service_namespace: value
        })
        let l = []
        res.data.forEach(item => {
            l.push({
                label: item.method + ' - ' + item.uri,
                value: item.name
            })
        })
        this.setState({
            apiList: l
        })
        if (l.length > 0) {
            this.form.setFieldsValue({
                api: l[0].value
            })
        }
    }
    onFinish = async (value: any) => {
        let data = {
            path: value.path,
            belong: value.belong,
            router_type: value.router_type,
            keep_type: value.keep_type,
            extension: JSON.stringify(this.state.extension),
            config: JSON.stringify(this.state.config),
            response: JSON.stringify(this.state.response)
        }
        this.props.onFinish(data)
    }
    setFormValue = (value) => {
        if (value.router_type == 'mesh') {
            this.setState({
                configJSON: true
            })
            this.form.setFieldsValue({
                path: value.path,
                belong: value.belong,
                router_type: value.router_type,
                keep_type: value.keep_type,
                description: value.description,
            })
        } else {
            this.setState({
                configJSON: false,
            })
            let ss = value.config.split('/')
            this.form.setFieldsValue({
                path: value.path,
                belong: value.belong,
                router_type: value.router_type,
                keep_type: value.keep_type,
                description: value.description,
                service: ss[0],
                api: ss[1]
            })
        }
        if (value.extension != null && value.extension.length > 0) {
            let r = JSON.parse(value.extension)
            this.json1.setJSON(r)
        }

        if (value.router_type == 'mesh' && value.config != null && value.config.length > 0) {
            let r = JSON.parse(value.config)
            this.json2.setJSON(r)
        }
        if (value.response != null && value.response.length > 0) {
            let r = JSON.parse(value.response)
            this.json3.setJSON(r)
        }
    }
    render() {
        const { configJSON, apiList } = this.state
        return (
            <Form
                {...layout}
                name="basic"
                initialValues={{ remember: true, keep_type: 'forever', router_type: 'mesh' }}
                onFinish={this.onFinish.bind(this)}
                onFinishFailed={() => { message.error('表单验证不通过') }}
                style={{ marginTop: "20px" }}
                ref={ele => this.form = ele}
            >
                <Form.Item
                    label="地址"
                    name="path"
                    rules={[{ required: true, message: '请输入' }]}
                >
                    <Input />
                </Form.Item>
                <Form.Item
                    label="分类"
                    name="belong"
                >
                    <Select style={{ width: '40%' }} options={this.belongList}>
                    </Select>
                </Form.Item>

                <Form.Item
                    label="类型"
                    name="router_type"
                >
                    <Radio.Group onChange={(e) => { this.setState({ configJSON: e.target.value == 'mesh' }) }}>
                        <Radio value={'mesh'}>多API聚合</Radio>
                        <Radio value={'relay'}>单API代理</Radio>
                    </Radio.Group>
                </Form.Item>

                <Form.Item
                    label="类型"
                    name="keep_type"
                >
                    <Radio.Group>
                        <Radio value={'forever'}>永久</Radio>
                        <Radio value={'tmp'}>临时路由（可删除）</Radio>
                    </Radio.Group>
                </Form.Item>

                <Form.Item
                    label="描述"
                    name="description"
                >
                    <TextArea rows={4} />
                </Form.Item>

                <Form.Item
                    label="扩展信息"
                    name="extension"
                >
                    <JSONEditor
                        json={{}}
                        onValidate={(val) => this.setState({ extension: val })}
                        height="250px"
                        ref={(ele) => { this.json1 = ele }}
                    />
                </Form.Item>

                <Form.Item
                    label="Request(mesh)"
                    name="config"
                    className={configJSON ? '' : 'hidden'}
                >
                    <JSONEditor
                        json={[[]]}
                        onValidate={(val) => { this.setState({ config: val }) }}
                        height="400px"
                        ref={(ele) => { this.json2 = ele }}
                    />
                </Form.Item>

                <Form.Item
                    label="Request(relay)"
                    className={configJSON ? 'hidden' : ''}
                >
                    <Form.Item
                        name="service"
                    >
                        <Select style={{ width: '60%' }} options={this.serviceList} onChange={this.changeService}></Select>
                    </Form.Item>
                    <Form.Item
                        name="api"
                    >
                        <Select style={{ width: '60%' }} options={apiList}></Select>
                    </Form.Item>
                </Form.Item>

                <Form.Item
                    label="Response"
                    name="response"
                >
                    <JSONEditor
                        json={{}}
                        onValidate={(val) => { this.setState({ response: val }) }}
                        height="400px"
                        ref={(ele) => { this.json3 = ele }}
                    />
                </Form.Item>

                <Form.Item {...tailLayout}>
                    <Button type="primary" htmlType="submit">
                        {this.props.triggerText}
                    </Button>
                </Form.Item>
            </Form>
        );
    }
}

export default FormX;
```

