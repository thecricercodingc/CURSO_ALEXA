# CURSO_ALEXA
**PLANTILLAS**

*Plantilla RequestIntentHandler*

# Auto-delegación
```
const INTENTNAMEIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'INTENTNAMEIntent';
    },
    handle(handlerInput) {
        //Tú Lógica
        //...
        //Fin Lógica
        //Construyendo diálogo de salida
        const speakOutput = `SALIDA`;
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt(handlerInput.t('HELP_MSG'))
            .getResponse();
    }
};
```
# Delegación manual
```
const INTENTNAMEIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'INTENTNAMEIntent';
    },
    handle(handlerInput) {
        //Tú Lógica
        //...
        //Fin Lógica
        //Delegación de directivas
        if(Alexa.getDialogState(handlerInput.requestEnvelope) === 'STARTED'){ 
            const CURRENTINTETN = handlerInput.requestEnvelope.request.intent;       
            let SLOTNAME = CURRENTINTETN.slots.SLOTNAME;
            if (!SLOTNAME.value) {
              CURRENTINTETN.slots.SLOTNAME.value = DEFAULTVALUE;
            }
            return handlerInput.responseBuilder
              .addDelegateDirective(CURRENTINTETN)
              .getResponse();
        }else if(Alexa.getDialogState(handlerInput.requestEnvelope) === 'IN_PROGRESS'){
            return handlerInput.responseBuilder
                .addDelegateDirective() 
                .getResponse();
        }else{//Construcción de diálogo de salida
            //Construyendo diálogo de salida
            const {dialog} = handlerInput.t('INTENTNAMEIntent');
            return handlerInput.responseBuilder
                .speak(dialog.Completed)
                .reprompt(handlerInput.t('HELP_MSG'))
                .getResponse();
        }
    }
};
```


# Combinación de Delegación manual y Auto-delegación
```
const INTENTNAMEIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'INTENTNAMEIntent';
    },
    handle(handlerInput) {
        //Tú Lógica
        //...
        //Fin Lógica
        const {SLOTNAME} = handlerInput.requestEnvelope.request.intent.slots;
        //Definiendo reglas para la directiva del turno
        if(!SLOTNAME.value){//Elicitación
            const {Elicitation} = handlerInput.t('INTENTNAMEIntent', {VARIABLE: VALOR).promps;
            return handlerInput.responseBuilder
    			.speak(Elicitation.SLOTNAME)
    			.addElicitSlotDirective(SLOTNAME.name)
    			.getResponse();
        }else if(SLOTNAME.value > NUMBER){//Validación
            const {Validation} = handlerInput.t('INTENTNAMEIntent').promps;
            return handlerInput.responseBuilder
    			.speak(Validation.SLOTNAME)
    			.addElicitSlotDirective(SLOTNAME.name)
    			.getResponse();
        }else if(SLOTNAME.confirmationStatus === 'NONE'){//Confirmación
            const {Confirmation} = handlerInput.t('INTENTNAMEIntent').promps;
            return handlerInput.responseBuilder
    			.speak(Confirmation.None.SLOTNAME)
    			.addConfirmSlotDirective(SLOTNAME.name)
    			.getResponse();        
        }else if(SLOTNAME.confirmationStatus === 'DENIED'){//Confirmación
            const {Confirmation} = handlerInput.t('INTENTNAMEIntent').promps;
            return handlerInput.responseBuilder
    			.speak(Confirmation.Denied.SLOTNAME)
    			.addElicitSlotDirective(SLOTNAME.name)
    			.getResponse();
        }else if(Alexa.getDialogState(handlerInput.requestEnvelope) !== 'COMPLETED'){//Delegación de directivas
            return handlerInput.responseBuilder
                .addDelegateDirective() 
                .getResponse();
        }else{//Construcción de diálogo de salida
            //Construyendo diálogo de salida
            const {dialog} = handlerInput.t('INTENTNAMEIntent');
            return handlerInput.responseBuilder
                .speak(dialog.Completed)
                .reprompt(handlerInput.t('HELP_MSG'))
                .getResponse();
        }
    }
};
```
