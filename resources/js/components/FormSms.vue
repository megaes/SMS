<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-10">
                <textarea class="form-control" id="txt" maxlength="4096" v-model="text"></textarea>
            </div>
            <div class="col-2">
                <div class="custom-control custom-checkbox">
                    <input type="checkbox" class="custom-control-input" id="customCheck" @click="onTransliterate">
                    <label class="custom-control-label noselect" for="customCheck">Транслитерировать</label>
                </div>
                <div class="mt-1 noselect"> Количество символов: {{ countSymbol }} </div>
                <div class="mt-1 noselect"> Количество SMS: {{ countSMS }} </div>
                <button type="button" class="btn btn-primary mt-2" :disabled="countSymbol ? false : true" @click="onSave">
                    Сохранить
                </button>
            </div>
        </div>
    </div>
</template>

<script>
    import dict from '../dictionary';

    export default
    {
        data()
        {
            return {
                text: '',
                isTransliterated: false
            };
        },
        computed:
        {
            countSymbol: function ()
            {
                return this.text.length;
            },
            countSMS: function ()
            {
                const lenSMS = this.isTransliterated ? 160 : 70;
                const lenPackedSMS = this.isTransliterated ? 153 : 67;
                const lenText = this.countSymbol;
                return lenText ? ((lenText > lenSMS) ? Math.ceil(lenText / lenPackedSMS) : 1) : 0;
            },
        },
        methods:
        {
            onTransliterate()
            {
                this.isTransliterated = !this.isTransliterated;

                const indKey    = this.isTransliterated ? 0 : 1;
                const indValue  = this.isTransliterated ? 1 : 0;

                let text = '';
                for(let pos = 0; pos < this.countSymbol;)
                {
                    let indMatch = -1;
                    let maxLength = 0;
                    dict.forEach((item, index) =>
                    {
                        const keyLength = item[indKey].length;
                        if ((maxLength < keyLength) && (keyLength <= this.countSymbol - pos))
                        {
                            let i = 0;
                            for(; (i < keyLength) && (this.text[pos + i] == item[indKey][i]); ++i);
                            if (i == keyLength)
                            {
                                maxLength = keyLength;
                                indMatch = index;
                            }
                        }

                    });
                    text += maxLength ? dict[indMatch][indValue] : this.text[pos];
                    pos += maxLength ? maxLength : 1;
                }
                this.text = text;
            },
            onSave()
            {
                axios.post('/addMessage',
                {
                    text: this.text,
                    countSMS: this.countSMS
                });
            }
        }
    }
</script>

<style lang="scss">
    .noselect
    {
        user-select: none;
    }
    #txt
    {
        resize: none;
        height: 150px !important;
    }
</style>