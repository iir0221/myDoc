# Event and Listener Model

事件机制

一个简单的例子：值更改事件，使用onchange特性

```html

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"  
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:f="http://java.sun.com/jsf/core"
      xmlns:h="http://java.sun.com/jsf/html">
   <h:head>
      <h:outputStylesheet library="css" name="styles.css"/>
      <title>#{msgs.windowTitle}</title>
   </h:head>

   <h:body>
      <h:form>
         <span class="emphasis">#{msgs.pageTitle}</span>
         <h:panelGrid columns="2">
            #{msgs.streetAddressPrompt}
            <h:inputText value="#{form.streetAddress}"/>

            #{msgs.cityPrompt}
            <h:inputText value="#{form.city}"/>

            #{msgs.statePrompt}
            <h:inputText value="#{form.state}"/>

            #{msgs.countryPrompt} 
            <h:selectOneMenu value="#{form.country}" onchange="submit()"
                  valueChangeListener="#{form.countryChanged}">
               <f:selectItems value="#{form.countries}" var="loc"
                  itemLabel="#{loc.displayCountry}" itemValue="#{loc.country}"/>
            </h:selectOneMenu>
         </h:panelGrid>
         <h:commandButton value="#{msgs.submit}"/>
      </h:form>
   </h:body>
</html>

```
debug：

事件发生阶段：processValidators以后
ProcessValidators.java

```java
    public void execute(FacesContext facesContext) throws FacesException {

        if (LOGGER.isLoggable(Level.FINE)) {
            LOGGER.fine("Entering ProcessValidationsPhase");
        }
        UIComponent component = facesContext.getViewRoot();
        assert (null != component);

        try {
            component.processValidators(facesContext);
        } catch (RuntimeException re) {
            String exceptionMessage = re.getMessage();
            if (null != exceptionMessage) {
                if (LOGGER.isLoggable(Level.WARNING)) {
                    LOGGER.log(Level.WARNING, exceptionMessage, re);
                }
            }
            throw new FacesException(exceptionMessage, re);
        }
        if (LOGGER.isLoggable(Level.FINE)) {
            LOGGER.fine("Exiting ProcessValidationsPhase");
        }

    }

```
UIViewRoot.java

```java
public void processValidators(FacesContext context) {
        initState();
        notifyBefore(context, PhaseId.PROCESS_VALIDATIONS);

        try {
            if (!skipPhase) {
                if (context.getPartialViewContext().isPartialRequest() &&
                    !context.getPartialViewContext().isExecuteAll()) {
                    context.getPartialViewContext().processPartial(PhaseId.PROCESS_VALIDATIONS);
                } else {
                    super.processValidators(context);
                }
                // 向事件源广播事件
                broadcastEvents(context, PhaseId.PROCESS_VALIDATIONS);
            }
        } finally {
            clearFacesEvents(context);
            notifyAfter(context, PhaseId.PROCESS_VALIDATIONS);
        }
    }
```

UIViewRoot.java
```java
public void broadcastEvents(FacesContext context, PhaseId phaseId) {

        if (null == events) {
            // no events have been queued
            return;
        }
        boolean hasMoreAnyPhaseEvents;
        boolean hasMoreCurrentPhaseEvents;

        // 获取相关的事件
        List<FacesEvent> eventsForPhaseId =
             events.get(PhaseId.ANY_PHASE.getOrdinal());

        // keep iterating till we have no more events to broadcast.
        // This is necessary for events that cause other events to be
        // queued.  PENDING(edburns): here's where we'd put in a check
        // to prevent infinite event queueing.
        do {
            // broadcast the ANY_PHASE events first
            if (null != eventsForPhaseId) {
                // We cannot use an Iterator because we will get
                // ConcurrentModificationException errors, so fake it
                // 遍历事件
                while (!eventsForPhaseId.isEmpty()) {
                    // 本例的事件是ValueChangeEvent
                    FacesEvent event =
                          eventsForPhaseId.get(0);
                    // 获取事件源，本例中是HtmlSelectOneMenu的实例
                    UIComponent source = event.getComponent();
                    UIComponent compositeParent = null;
                    try {
                        if (!UIComponent.isCompositeComponent(source)) {
                            compositeParent = UIComponent.getCompositeComponentParent(source);
                        }
                        if (compositeParent != null) {
                            compositeParent.pushComponentToEL(context, null);
                        }
                        source.pushComponentToEL(context, null);
                        // 事件源向监听器发起广播
                        source.broadcast(event);
                    } catch (AbortProcessingException e) {
                        context.getApplication().publishEvent(context,
                                                              ExceptionQueuedEvent.class,
                                                              new ExceptionQueuedEventContext(context,
                                                                                        e,
                                                                                        source,
                                                                                        phaseId));
                    }
                    finally {
                        source.popComponentFromEL(context);
                        if (compositeParent != null) {
                            compositeParent.popComponentFromEL(context);
                        }
                    }
                    eventsForPhaseId.remove(0); // Stay at current position
                }
            }

            // then broadcast the events for this phase.
            if (null != (eventsForPhaseId = events.get(phaseId.getOrdinal()))) {
                // We cannot use an Iterator because we will get
                // ConcurrentModificationException errors, so fake it
                while (!eventsForPhaseId.isEmpty()) {
                    FacesEvent event = eventsForPhaseId.get(0);
                    UIComponent source = event.getComponent();
                    UIComponent compositeParent = null;
                    try {
                        if (!UIComponent.isCompositeComponent(source)) {
                            compositeParent = getCompositeComponentParent(source);
                        }
                        if (compositeParent != null) {
                            compositeParent.pushComponentToEL(context, null);
                        }
                        source.pushComponentToEL(context, null);
                        source.broadcast(event);
                    } catch (AbortProcessingException ape) {
                        // A "return" here would abort remaining events too
                        context.getApplication().publishEvent(context,
                                                              ExceptionQueuedEvent.class,
                                                              new ExceptionQueuedEventContext(context,
                                                                                        ape,
                                                                                        source,
                                                                                        phaseId));
                    }
                    finally {
                        source.popComponentFromEL(context);
                        if (compositeParent != null) {
                            compositeParent.popComponentFromEL(context);
                        }
                    }
                    eventsForPhaseId.remove(0); // Stay at current position
                }
            }

            // true if we have any more ANY_PHASE events
            hasMoreAnyPhaseEvents =
                  (null != (eventsForPhaseId =
                        events.get(PhaseId.ANY_PHASE.getOrdinal()))) &&
                        !eventsForPhaseId.isEmpty();
            // true if we have any more events for the argument phaseId
            hasMoreCurrentPhaseEvents =
                  (null != events.get(phaseId.getOrdinal())) &&
                  !events.get(phaseId.getOrdinal()).isEmpty();

        } while (hasMoreAnyPhaseEvents || hasMoreCurrentPhaseEvents);
    
    }

```
本例的事件源是HtmlSelectOneMenu实例，调用其父类UIComponentBase的方法向监听器发起广播

UIComponentBase.java
```java
    public void broadcast(FacesEvent event)
        throws AbortProcessingException {

        if (event == null) {
            throw new NullPointerException();
        }
        if (event instanceof BehaviorEvent) {
            BehaviorEvent behaviorEvent = (BehaviorEvent) event;
            Behavior behavior = behaviorEvent.getBehavior();
            behavior.broadcast(behaviorEvent);
        }

        if (listeners == null) {
            return;
        }

        // 遍历注册在该事件源上的监听器
        for (FacesListener listener : listeners.asArray(FacesListener.class)) {
            // 如果是合适的监听器
            if (event.isAppropriateListener(listener)) {
                // 执行事件监听器，本例的事件是ValueChangeEvent
                event.processListener(listener);
            }
        }
    }
```

ValueChangeEvent.java
```java
public void processListener(FacesListener listener) {

        ((ValueChangeListener) listener).processValueChange(this);

    }

```

最终调用在页面中绑定的
```html
valueChangeListener="#{form.countryChanged}"

```

对值作出改变

