# React & ant.design 使用小结

1. **列表元数据中的key不能使用索引**

  具体细节参见文章：

  > [React 实践心得：key 属性的原理和用法](http://taobaofed.org/blog/2016/08/24/react-key/);
  > [index作为key是反模式](https://segmentfault.com/a/1190000006152449);

  代码中可使用[shortid](https://github.com/dylang/shortid)类库生成随机ID

1. **控件校验规则**

  Ant.desgin提供里控件校验规则功能，但必须作为[Form](https://github.com/ant-design/ant-design/blob/master/components/form/Form.tsx)的子元素才能生效。下面以一段代码示例

  常见代码

    ```javascript
      import React from "react";
      import {Form, Row, Col, Select} from "antd";

      const Option = Select.Option;
      const formItemLayout = {
        labelCol: {
          xs: { span: 24 },
          sm: { span: 7 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 12 },
          md: { span: 10 },
        },
      };

      @Form.create()
      export default class OrderForm extends React.Component {
        handleSubmit = () => {
          this.props.form.validateFieldsAndScroll((errors, values) => {
            if (!errors) {
              //do something
            }
          });
        }
        render(){
          const {getFieldDecorator} = this.props.form;

          return (
            <Form>
              <Row>
                <Col span={4}>
                  <Form.Item label={'BG'}
                    {...FormItemLayout}>
                      {getFieldDecorator('BG', {
                        rules: [{
                          required: true, message: '请选择BG',
                        }],
                      })(
                        <Select>
                          <Option key='1' value='WXG'>WXG</Option>
                        </Select>
                      )}
                  </Form.Item>
                </Col>
                <Col span={4}>
                  <Form.Item label={'XBG'}
                    {...FormItemLayout}>
                      {getFieldDecorator('XBG', {
                        rules: [{
                          required: true, message: '请选择虚拟BG',
                        }],
                      })(
                        <Select>
                          <Option key='1' value='XWXG'>XWXG</Option>
                        </Select>
                      )}
                  </Form.Item>
                </Col>
              </Row>
              <Row>
                <Button onClick={this.handleSubmit}>提交</Button>
              </Row>
            </Form>
          );
        }
      }
    ```

  以上代码存在以下问题:
    - Selet等控件没有复用
    - 如果同时存在多个form，form submit时将触发其他form的校验规则。因为类中所有components props都是指向同一个Form实例，即装饰器Form.create()
    - 代码耦合紧，影响后续维护

  针对以上问题，优化代码如下
  > 封装Select控件

    ```javascript
    import React from "react";
    import { Select } from "antd";
    import shortid from "shortid"
    import FormItem from "../FormItem";

    const Option = Select.Option;

    export default class SelectList extends React.Component {
      render() {
        const { getFieldDecorator } = this.props.form;
        const { id, label, require, data } = this.props;

        return (
          <FormItem label={label}>
            {getFieldDecorator(`${id}`, {
              rules: [{ required: { require }, message: `请选择${label}` }],
            })(
              <Select>
                {
                  data.map(item => {
                    return <Option key={shortid.generate()}
                              value={`${item.code}`}>
                              {`${item.name}`}
                            </Option>;
                  })
                }
              </Select>
              )}
          </FormItem>
        );
      }
    }
    ```
  > 提取样式，封装到FormItem中

    ```javascript
    import React from "react";
    import { Form, Col } from "antd";

    const formItemLayout = {
      labelCol: {
        xs: { span: 24 },
        sm: { span: 7 },
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 12 },
        md: { span: 10 },
      },
    };

    export default class FormItem extends React.Component {
      render() {
        const { children, ...others } = this.props;
        return (
          <Col span={6}>
            <Form.Item {...formItemLayout}
              //将外部传递的props放在最后，这样如果存在相同属性时将覆盖前面的值，方便扩展
              {...others}>
              {children}
            </Form.Item>
          </Col>
        );
      }
    }
    ```
  > 封装Form，通过递归将Form实例参数form传递给每个控件，将校验规则指向到正确的form实例

    ```javascript
    import React from "react";
    import { Form } from "antd";

    @Form.create()
    export default class DecoratorForm extends React.Component {
      cloneDetailComponents = (children, form) => {
        return React.Children.map(children, child => {
          const subChildren = child.props.children;
          //递归克隆元素，并将form作为props递归给每个控件
          return React.cloneElement(child,
            { form: form },
            this.cloneDetailComponents(subChildren, form));
        });
      };

      render() {
        const { children, form } = this.props;
        const cloneChildren = this.cloneDetailComponents(children, form);

        return (
          <Form>
            {cloneChildren}
          </Form>
        );
      }
    }
    ```
  > 提取Submit Button

    ```javascript
    import React from "react";
    import { Button } from "antd";

    export default class Submit extends React.Component {
      handleSubmit = () => {
        this.props.form.validateFieldsAndScroll((errors, values) => {
          if (!errors) {
            //do something
          }
        });
      }

      render() {
        return (
          <Col span={24}>
            <Button onClick={this.handleSubmit}>提交</Button>
          </Col>
        );
      }
    }
    ```
  > 最终页面

    ```javascript
    import React from 'react';
    import ReactDOM from 'react-dom';
    import SelectList from './components/SelectList';
    import DecoratorForm from "./components/DecoratorForm";
    import Submit from "./components/Submit";
    import {Row} from "antd";

    const bgProps  = {
        id:'bg',
        label:'BG',
        require:true,
        data:[{
            code:'WXG',
            name:'WXG'
        }]
    };
    const xbgProps  = {
        id:'xbg',
        label:'虚拟BG',
        require:true,
        data:[{
            code:'XWXG',
            name:'XWXG'
        }]
    };

    ReactDOM.render((
        <DecoratorForm>
            <Row>
                <SelectList {...bgProps}/>
                <SelectList {...xbgProps}/>
            </Row>
            <Row>
                <Submit/>
            </Row>
        </DecoratorForm>
    ), document.getElementById('root'));
    ```

1. **Higher-Order Component (HOC)**
   参考文章
   >  [深入理解 React 高阶组件](https://zhuanlan.zhihu.com/p/24776678)