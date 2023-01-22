<script setup>
import {ref, computed} from 'vue';

// Переменные

const data = {
    num0: 0,
    num1: 1,
    num2: 2,
    num3: 3,
    num4: 4,
    num5: 5,
    num6: 6,
    num7: 7,
    num8: 8,
    num9: 9,
    clearAll: 'C',
    delete: 'del',
    clearCurrent: 'CE',
    divide: '÷',
    mult: '×',
    minus: '-',
    plus: '+',
    equal: '=',
}

const zero = [0, '0'];

const operators = ['+', '-', '÷', '×'];

const allSymbols = ['÷', 7, 8, 9, '×', 4, 5, 6, '-', 1, 2, 3, '+', 0];

const reg = new RegExp('^[0-9]$');

// Реактивные переменные

const currentValue = ref('');

const currentValueArray = computed(() => {
    return currentValue.value.split('');
})

const operationValue = ref('');

const operationValueArray = computed(() => {
   return operationValue.value.split('')
});

const calculatorStatus = ref(false);

const checkLastSymbol = computed(() => {
    return compareElementInArray(searchLastSymbolInArray(operationValueArray.value), operators)
});


// Функции



const isNumber = (symbol) => {
    return reg.test(symbol);
}

const symbolOrNumberWrapper = (symbol) => {
    if (isNumber(symbol)) {
        return clickNumber(symbol)
    } else {
        return clickOperator(symbol)
    }
}

const searchLastSymbolInArray = (array) => {
        return array[array.length - 1];
};

const compareElementInArray = (elementToCompare, array) => {
    return array.some((elem) => elem === elementToCompare)
}

const changeLastSymbol = (array, newItem) => {
    array.splice(array.length -1, 1, newItem)
    return array.join('');
}

const replacedValue = (string) => {
    return string.replaceAll('÷', '/').replaceAll('×', '*')
}

const changeDisplay = (value) => {
    changeOperationValue(value);
    changeCurrentValue(value);
}

const deleteSymbolInOperation = () => {
    operationValueArray.value.pop();
    operationValue.value = operationValueArray.value.join('');
}

const deleteSymbolInCurrent = () => {
    currentValueArray.value.pop();
    currentValue.value = currentValueArray.value.join('')

}

const deleteBtn = () => {

    if (calculatorStatus.value){
        clearAll();
    }
    else {
        deleteSymbolInOperation()
        deleteSymbolInCurrent()

    }
}

// Основной функционал

const changeOperationValue = (num) => {
    let firstSymbol = operationValue.value[0];

    if (
        compareElementInArray(firstSymbol, operators) ||
        compareElementInArray(firstSymbol, zero)
    ) {
        console.log(operationValue.value);

        operationValue.value = '';
    };

    if (
        checkLastSymbol.value &&
        compareElementInArray(num, operators)
    ) {
        console.log(checkLastSymbol.value)
        operationValue.value = changeLastSymbol(operationValueArray.value, num)
    } 
    else {
        operationValue.value += num;
    }

}

const changeCurrentValue = (num) => {


    if (isNumber(num)) {
        currentValue.value += num
    } else {
        currentValue.value = num;
    }

    let firstSymbol = currentValue.value.split('')[0];


    if (
        compareElementInArray(firstSymbol, operators) ||
        compareElementInArray(firstSymbol, zero) &&
        isNumber(num)
    ) {
        currentValue.value = '';
        currentValue.value += num
    }

}

const clickOperator = (value) => {

    if (calculatorStatus.value) {
        calculatorStatus.value = false;
        operationValue.value = currentValue.value + value;
        changeDisplay(value);
    } else {
        changeDisplay(value)
    }

}

const clickNumber = (value) => {

    if (calculatorStatus.value) {
     
        
        currentValue.value = '';
        operationValue.value = '';
        calculatorStatus.value = false;
        changeDisplay(value)
    
    } else {
        changeDisplay(value)

    }

}

const getResult = () => {
    
    if (checkLastSymbol.value) {
        currentValue.value = changeLastSymbol(operationValueArray.value)
    } else {
        currentValue.value = eval(replacedValue(operationValue.value))
        calculatorStatus.value = true;
    }

}

const clearAll = () => {
    operationValue.value = '';
    currentValue.value = '0';
}

const clearCurrentValue = () => {

    if (calculatorStatus.value) {
        clearAll();
    } 
    else if(currentValue.value === '0') {

    } 
    else {

        let lengthDifference = operationValueArray.value.length - currentValueArray.value.length
        let newOperationArray = operationValueArray.value.slice(0, lengthDifference);

        operationValue.value = newOperationArray.join('');
        currentValue.value = '0';
    }

 
}




</script> 



<template>
    <div class="wrapper">
        
            <div class="calc">
                <div class="calc__container">
                    <div class="calc__display">
                        <div class="display-operation-value">{{ operationValue }}</div>
                        <div class="display-current-value">{{ currentValue }}</div>
                    </div>
                    <div class="calc__content">
                        <div class="button button_symbol" @click="clearAll">{{ data.clearAll }}</div>
                        <div class="button button_symbol" @click="clearCurrentValue">{{ data.clearCurrent }}</div>
                        <div class="button button_symbol" @click="deleteBtn">{{ data.delete }}</div>
                        <div
                        :class="[
                            'button',
                            { 'button_numbers' : isNumber(symbol)},
                            {'button_symbol' : !isNumber(symbol)},
                        ]"
                        @click="symbolOrNumberWrapper(symbol)"
                        v-for="symbol in allSymbols"
                        :key="symbol">
                            {{ symbol }}
                        </div>
                        <div class="button button_equal" @click="getResult">{{ data.equal }}</div>
                    </div>
                </div>
            </div>
        
       
    </div>
</template>

<style scoped lang="scss">

.wrapper {
    width: 100%;
    height: 100vh;
    display: grid;
    place-items: center;
}


.calc {
    width: 50%;
    height: 70%;
  

    &__container {
        height: 100%;
        padding: 10px;
        background-color: var(--calc-bg);
    }

    &__display {
    background-color: var(--disp-bg);
    color: var(--symbols);
    min-height: 20%;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    justify-content: space-around;
    width: 100%;
    text-align: end;

    .display-operation-value {
        color: var(--operation-value);
        font-size: 0.7em;
        margin-right: 10px;
    }
    .display-current-value {
        margin-right: 10px;

        color: inherit;
    }
}
    &__content {
    background-color: var(--calc-bg);
    height: 80%;
    color: white;
    display: grid;
    width: 100%;
    grid-template-columns: repeat(4, 1fr);
    grid-template-rows: repeat(5, 1fr);
    grid-column-gap: 5px;
    grid-row-gap: 5px;
    place-items: center;
    margin: 5px 0px 5px 0px;

    :nth-child(17) {
        grid-area: 5 / 2 / 6 / 3;
    }
    :nth-child(18) {
        grid-area: 5 / 4 / 6 / 5;

    }
    }
}

.button {
    border-radius: 10px;
    display: grid;
    place-items: center;
    width: 100%;
    height: 100%;
    cursor: pointer;



    &_numbers {
        background-color: var(--calc-buttons-numbers);

        &:hover {
            background-color: var(--calc-buttons-operators);
        }
    }
    &_symbol {
        background-color: var(--calc-buttons-operators);

        &:hover {
            background-color: var(--calc-buttons-numbers);
        }
    }
    &_equal {
        background-color: var(--calc-button-equal);

        &:hover {
            background-color: var(--calc-button-equal-hovered);
        }
    }
}

@media screen and (max-width: 2560px) {
    .wrapper {
        font-size: 1.2em;
    }
}
@media screen and (max-width: 1920px) {
    .wrapper {
        font-size: 1em;
    }
}
@media screen and (max-width: 1440px) {
    .wrapper {
        font-size: 0.9em;
    }
}
@media screen and (max-width: 1024px) {
    .wrapper {
        font-size: 0.8em;
    }
}
@media screen and (max-width: 768px) {
    .wrapper {
        font-size: 0.6em;
    }
}
@media screen and (max-width: 425px) {
    .wrapper {
        font-size: 0.5em;
    }
}
@media screen and (max-width: 375px) {
    .wrapper {
        font-size: 0.5em;
    }
}
@media screen and (max-width: 320px) {
    .wrapper {
        font-size: 0.4em;
    }
}


</style>