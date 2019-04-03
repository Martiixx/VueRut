<template>
    <div class="row">
        <div class="col-md-6">
        <fg-input label="Rut: *"
                    name="rut"
                    v-model="model.rut"
                    v-on:blur="checkExist"
                    v-on:keyup="formatRut"
                    v-validate="validations.rut"
                    :error="getError('rut')"
                    placeholder="Ingrese Rut">
        </fg-input>
        <small class="text text-danger" v-show="showRutInvalido">El Rut es invalido</small>
        <small class="text text-danger" v-show="showRutError">El Rut ingresado ya está registrado en el sistema</small>
        </div>
    </div>
</template>
<script>
    import axios from 'axios'
    
    export default {
        data () {
            return {
                model: {
                    rut: ''
                },
                showRutInvalido: false,
                showRutError: false,
                validations: {
                    rut: {
                        regex: '^0*(\\d{1,3}(\\.?\\d{3})*)\\-?([\\dkK])$',
                        required: true,
                        min: 8,
                        max: 12
                    }
                }
            }
        },
        methods: {
            getError (fieldName) {
                return this.errors.first(fieldName)
            },
            validate () {
                return this.$validator.validateAll().then(res => {
                return res
                })
            },
            checkExist () {
                let rut = this.model.rut
                // Petición al backend para verificar si existe el rut, la idea es retornar un true o false desde el backend
                this.$http.get(url + '/verificar-rut?rut=' + rut).then(response => { // peticion con vue-resource
                    this.showRutError = response
                    return this.showRutError
                }).catch((error) => {
                    console.log(error)
                })
                
                axios.get(url + '/verificar-rut?rut=' + rut)then(response => { // Petición con axios
                    this.showRutError = response
                    return this.showRutError
                }).catch((error) => {
                    console.log(error)
                })
            },
            formatRut () {
                let clearRut = typeof this.model.rut === 'string' ? this.model.rut.replace(/[^0-9kK]+/g, '').toUpperCase() : '' // limpiamos la variable rut
                if (clearRut.length <= 1) {
                    return false
                }
                var result = clearRut.slice(-4, -1) + '-' + clearRut.substr(clearRut.length - 1)
                for (var i = 4; i < clearRut.length; i += 3) {
                    result = clearRut.slice(-3 - i, -i) + '.' + result
                }
                this.model.rut = result

                let rut = this.model.rut

                if (typeof rut !== 'string') {
                    return false
                }
                rut = typeof this.model.rut === 'string' ? this.model.rut.replace(/[^0-9kK]+/g, '').toUpperCase() : ''
                var rutDigits = parseInt(rut.slice(0, -1), 10)
                var m = 0
                var s = 1
                while (rutDigits > 0) {
                    s = (s + rutDigits % 10 * (9 - m++ % 6)) % 11
                    rutDigits = Math.floor(rutDigits / 10)
                }
                var checkDigit = (s > 0) ? String((s - 1)) : 'K'
                if (checkDigit === rut.slice(-1)) {
                    this.showRutInvalido = false
                } else {
                    this.showRutInvalido = true
                }
            }
        }
        
    }
</script>
  
