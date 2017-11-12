# Restore View. 
* 恢复UIComponent tree
# Apply Request Values. 
* 可编辑的Component应该实现EditableValueHolder接口
* 遍历Component tree，每个Component会调用processDecodes()方法。对于大多数Component来说(除UIData等复杂的Component)，接着调用decode()方法。
```java
public void processDecodes(FacesContext context) {

    if (context == null) {
        throw new NullPointerException();
    }

    // Skip processing if our rendered flag is false
    if (!isRendered()) {
        return;
    }

    pushComponentToEL(context, null);

    try {
        // Process all facets and children of this component
        Iterator kids = getFacetsAndChildren();
        while (kids.hasNext()) {
            UIComponent kid = (UIComponent) kids.next();
            kid.processDecodes(context);
        }

        // Process this component itself
        try {
            decode(context);
        } catch (RuntimeException e) {
            context.renderResponse();
            throw e;
        }
    } finally {
        popComponentFromEL(context);
    }
}
```
* decode()方法找到Component对应的renderer,并且调用renderer的decode(context,this)方法（会将自己作为参数传递给该方法）

```java
    public void decode(FacesContext context) {

    if (context == null) {
        throw new NullPointerException();
    }
    String rendererType = getRendererType();
    if (rendererType != null) {
        Renderer renderer = this.getRenderer(context);
        if (renderer != null) {
            renderer.decode(context, this);
        } else {
            if (LOGGER.isLoggable(Level.FINE)) {
                LOGGER.fine("Can't get Renderer for type " + rendererType);
            }
        }
    }
}
```
* renderer的decode()方法会调用setSubmittedValue()方法。
```java
@Override
public void decode(FacesContext context, UIComponent component) {

    rendererParamsNotNull(context, component);

    if (!shouldDecode(component)) {
        return;
    }

    String clientId = decodeBehaviors(context, component);

    if (!(component instanceof UIInput)) {
        // decode needs to be invoked only for components that are
        // instances or subclasses of UIInput.
        if (logger.isLoggable(Level.FINE)) {
            logger.log(Level.FINE,
                        "No decoding necessary since the component {0} is not an instance or a sub class of UIInput",
                        component.getId());
        }
        return;
    }

    if (clientId == null) {
        clientId = component.getClientId(context);
    }

    assert(clientId != null);
    Map<String, String> requestMap =
            context.getExternalContext().getRequestParameterMap();
    // Don't overwrite the value unless you have to!
    String newValue = requestMap.get(clientId);
    if (newValue != null) {
        setSubmittedValue(component, newValue);
        if (logger.isLoggable(Level.FINE)) {
            logger.log(Level.FINE,
                        "new value after decoding {0}",
                        newValue);
        }
    }
}
```
* 将submitted value设置给Component的submittedValue属性。
```java
public void setSubmittedValue(UIComponent component, Object value) {

    if (component instanceof UIInput) {
        ((UIInput) component).setSubmittedValue(value);
        if (logger.isLoggable(Level.FINE)) {
            logger.fine("Set submitted value " + value + " on component ");
        }
    }

}
```
```java
public void setSubmittedValue(Object submittedValue) {
    this.submittedValue = submittedValue;
}
```
# Process Validations. 
* 这一阶段将调用processValidators()方法,然后调用validate()方法
```java
public void processValidators(FacesContext context) {

    if (context == null) {
        throw new NullPointerException();
    }

    // Skip processing if our rendered flag is false
    if (!isRendered()) {
        return;
    }

    pushComponentToEL(context, this);

    if (!isImmediate()) {
        Application application = context.getApplication();
        application.publishEvent(context, PreValidateEvent.class, this);
        executeValidate(context);
        application.publishEvent(context, PostValidateEvent.class, this);
    }
    for (Iterator<UIComponent> i = getFacetsAndChildren(); i.hasNext(); ) {
        i.next().processValidators(context);
    }

    popComponentFromEL(context);
}
```
```java
private void executeValidate(FacesContext context) {
    try {
        validate(context);
    } catch (RuntimeException e) {
        context.renderResponse();
        throw e;
    }

    if (!isValid()) {
        context.validationFailed();
        context.renderResponse();
    }
}
```

* validate()方法对submitted value进行convert（getConvertedValue(context, submittedValue);）和validates（validateValue(context, newValue);），然后调用setValue()方法。
```java
public void validate(FacesContext context) {

    if (context == null) {
        throw new NullPointerException();
    }

    // Submitted value == null means "the component was not submitted
    // at all".  
    Object submittedValue = getSubmittedValue();
    if (submittedValue == null) {
        if (isRequired() && isSetAlwaysValidateRequired(context)) {
            // continue as below
        } else {
            return;
        }
    }

    // If non-null, an instanceof String, and we're configured to treat
    // zero-length Strings as null:
    //   call setSubmittedValue(null)
    if ((considerEmptyStringNull(context)
            && submittedValue instanceof String 
            && ((String) submittedValue).length() == 0)) {
        setSubmittedValue(null);
        submittedValue = null;
    }

    Object newValue = null;

    try {
        newValue = getConvertedValue(context, submittedValue);
    }
    catch (ConverterException ce) {
        addConversionErrorMessage(context, ce);
        setValid(false);
    }

    validateValue(context, newValue);

    // If our value is valid, store the new value, erase the
    // "submitted" value, and emit a ValueChangeEvent if appropriate
    if (isValid()) {
        Object previous = getValue();
        setValue(newValue);
        setSubmittedValue(null);
        if (compareValues(previous, newValue)) {
            queueEvent(new ValueChangeEvent(this, previous, newValue));
        }
    }

}
```
* setValue()方法将通过convert和validate的newValue存储为local variable
```java
@Override
public void setValue(Object value) {
    super.setValue(value);
    // Mark the local value as set.
    setLocalValueSet(true);
}
```
```java
public void setValue(Object value) {
    getStateHelper().put(PropertyKeys.value, value);

}
```
* 如果local variable 不为null,其将通过getValue()方法获取
```java
    public Object getValue() {

    return getStateHelper().eval(PropertyKeys.value);

}
```
# Update Model Values. 
* 这一阶段调用processUpdates()方法,对于Input类Component，再调用updateModel()方法
```java
public void processUpdates(FacesContext context) {

    if (context == null) {
        throw new NullPointerException();
    }

    // Skip processing if our rendered flag is false
    if (!isRendered()) {
        return;
    }

    super.processUpdates(context);

    try {
        updateModel(context);
    } catch (RuntimeException e) {
        context.renderResponse();
        throw e;
    }

    if (!isValid()) {
        context.renderResponse();
    }
}
```
* 通过ValueExpression将value set到model中
```java
public void updateModel(FacesContext context) {

    if (context == null) {
        throw new NullPointerException();
    }

    if (!isValid() || !isLocalValueSet()) {
        return;
    }
    ValueExpression ve = getValueExpression("value");
    if (ve != null) {
        Throwable caught = null;
        FacesMessage message = null;
        try {
            ve.setValue(context.getELContext(), getLocalValue());
            resetValue();
        } catch (ELException e) {
            caught = e;
            String messageStr = e.getMessage();
            Throwable result = e.getCause();
            while (null != result &&
                    result.getClass().isAssignableFrom(ELException.class)) {
                messageStr = result.getMessage();
                result = result.getCause();
            }
            if (null == messageStr) {
                message =
                        MessageFactory.getMessage(context, UPDATE_MESSAGE_ID,
                            MessageFactory.getLabel(
                                context, this));
            } else {
                message = new FacesMessage(FacesMessage.SEVERITY_ERROR,
                                            messageStr,
                                            messageStr);
            }
            setValid(false);
        } catch (Exception e) {
            caught = e;
            message =
                    MessageFactory.getMessage(context, UPDATE_MESSAGE_ID,
                        MessageFactory.getLabel(
                            context, this));
            setValid(false);
        }
        if (caught != null) {
            assert(message != null);
            // PENDING(edburns): verify this is in the spec.
            @SuppressWarnings({"ThrowableInstanceNeverThrown"})
            UpdateModelException toQueue =
                    new UpdateModelException(message, caught);
            ExceptionQueuedEventContext eventContext =
                    new ExceptionQueuedEventContext(context,
                                            toQueue,
                                            this,
                                            PhaseId.UPDATE_MODEL_VALUES);
            context.getApplication().publishEvent(context,
                                                    ExceptionQueuedEvent.class,
                                                    eventContext);
            
        }
        
    }
}
```
# Invoke Application. 
* Button 事件监听器等将会在这个阶段被调用。 (as will navigation if memory serves).

# Render Response. 
* Tree通过renderers被rendered,state被保存。
* 如果上述任一阶段出现异常，将直接执行Render Phase

# 参考
[JSF Lifecycle and Custom components](https://stackoverflow.com/questions/33476/jsf-lifecycle-and-custom-components)