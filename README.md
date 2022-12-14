# CURSO_ALEXA
**IMPORTANTE**: *Recuerda añadir al final de este js en exports.handler toda RequestIntentHandler que crees.*

*Recuerda que para usar las directivas de diálogo (Ellicit, confirmation, etc) necesitarás hacer required los slots de la Intent, ver <a>https://github.com/alexa-samples/skill-sample-nodejs-decision-tree/issues/28</a>*

**Notas:**

* El órden de directivas por defecto ([Auto-delegación](#auto-delegación)) es Intent (Confirmation) y Slot (Elicitation, Confirmation y Validation).
* Si queremos cambiar el orden de required (en [Auto-delegación](#auto-delegación)) intent slot de una intent, ir al JSON del frontened, luego a dialog.intents y poner en órden que queremos que se muestren los required.
* El órden de directivas en [Delegación manual](#delegación-manual) y en la [Delegación combinada](#combinación-de-delegación-manual-y-auto-delegación) dependerá de cómo se organice la lógica en el código.
* Para entender correctamente [Delegación manual](#delegación-manual) y en la [Delegación combinada](#combinación-de-delegación-manual-y-auto-delegación), ver también [Plantilla promps](#plantilla-promps) 


## **Plantillas RequestIntentHandler**

### Auto-delegación
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
        const {dialog} = handlerInput.t('INTENTNAME', {VARIABLE: VARIABLE});
        return handlerInput.responseBuilder
            .speak(dialog.Completed)
            .reprompt(handlerInput.t('HELP_MSG'))
            .getResponse();
    }
};
```
### Delegación manual
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


### Combinación de Delegación manual y Auto-delegación
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
## Plantilla Promps 
```
INTENTNAMEIntent:{
    dialog: {
        Completed: 'Mensaje'
    },
    promps: {
        Elicitation: {
            SLOTNAME: 'Mensaje'
        },
        Confirmation: {
            None: {
                SLOTNAME: '¿Mensaje {{variable}}?',
            },
            Denied:{
                SLOTNAME: 'Mensaje'
            }
        },
        Validation: {
            SLOTNAME: 'Mensaje'
        }
    }
}
```

## Plantillas Acceso a APIS externas (con AXIOS)

### Método GET

```
getRemoteData (url) {
  /*Modo de uso
    const url = "https://anydir.com"
    let speakOutput = "No se cargaron los datos";
    await logic.getRemoteData(url)
        .then((data) => {
            speakOutput = data.results.toString();
        })
  */
  return new Promise((resolve, reject) => {
      axios
      .get(url)
      .then(res => {
        console.log(res.data);
        resolve(res.data);
      })
      .catch(error => {
        console.error(error);
      });
  }) 
},
```

### Método POST

```
postRemoteData (url) {
  /*Modo de uso
    const url = "https://anydir.com"
    let speakOutput = "No se cargaron los datos";
    await logic.postRemoteData(url, {param1: valueParam1, param2: valueParam2, ...})
        .then((data) => {
            speakOutput = data.results.toString();
        })
  */
  return new Promise((resolve, reject) => {
      axios
      .post(url)
      .then(res => {
        console.log(res.data);
        resolve(res.data); 
      })
      .catch(error => {
        console.error(error);
      });
  }) 
}
```
