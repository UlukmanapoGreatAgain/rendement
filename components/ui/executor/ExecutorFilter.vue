<template>
    <app-filter v-model="values"
                url="/api/getExecutors/"
                path-encoding
                :encoding-description="description"
                :count-endings="countEndings"
                @update:applied="$emit( 'update:applied', $event )"
                @apply="$emit( 'apply', $event )">
        <template v-slot:body="{ apply, clear, loading, countFormat, appliedValuesCount, onValuesChange, isFilterEmpty }">
            <v-row class="filter__body"
                   :class="isMobile ? { 'mt-8 mb-20': !isFilterEmpty, 'mb-20': isFilterEmpty } : {}">
                <!-- Категория -->
                <v-col cols="12" lg="4"
                       order="2" order-lg="1"
                       class="filter__field">
                    <!-- Десктоп: Категория -->
                    <category-select v-if="isDesktop"
                                     v-model="values.arCategoryCode"
                                     url="/api/categories/"
                                     :sub-categories.sync="subCategories"
                                     multiple hide-details
                                     @input="onValuesChange"/>

                    <!-- Мобайл -->
                    <v-list v-if="isMobile">
                        <!-- Категория -->
                        <v-list-item class="l-border bwb-1"
                                     @click="dialogs.category = true">
                            <v-list-item-content class="d-block subtitle-1">
                                <v-badge :value="values.arCategoryCode.length > 0"
                                         color="primary"
                                         inline
                                         :content="values.arCategoryCode.length">
                                    Категория
                                </v-badge>
                            </v-list-item-content>


                            <v-list-item-action>
                                <v-icon>mdi-chevron-right</v-icon>
                            </v-list-item-action>
                        </v-list-item>

                        <!-- Подкатегория -->
                        <v-list-item class="l-border bwb-1"
                                     :disabled="!values.arCategoryCode.length"
                                     @click="dialogs.subCategory = true">
                            <v-list-item-content class="d-block subtitle-1">
                                <v-badge :value="values.arSubcategoryId.length > 0"
                                         color="primary"
                                         inline
                                         :content="values.arSubcategoryId.length">
                                    Подкатегория
                                </v-badge>
                            </v-list-item-content>


                            <v-list-item-action>
                                <v-icon>mdi-chevron-right</v-icon>
                            </v-list-item-action>
                        </v-list-item>

                        <!-- Месторождение -->
                        <v-list-item class="l-border bwb-1"
                                     @click="dialogs.location = true">
                            <v-list-item-content class="d-block subtitle-1">
                                <v-badge :value="Number.isInteger( values.locationId )"
                                         color="primary"
                                         inline
                                         :content="1">
                                    Месторождение
                                </v-badge>
                            </v-list-item-content>

                            <v-list-item-action>
                                <v-icon>mdi-chevron-right</v-icon>
                            </v-list-item-action>
                        </v-list-item>
                    </v-list>
                </v-col>

                <!-- Подкатегория -->
                <v-col v-if="isDesktop"
                       cols="12" lg="4"
                       order="2"
                       class="filter__field">
                    <app-select v-model="values.arSubcategoryId"
                                :items="subCategories"
                                item-text="name" item-value="id"
                                hide-details multiple
                                label="Подкатегория"
                                no-data-text="Выберите категорию"
                                @input="onValuesChange"/>
                </v-col>

                <!-- Поиск по ключевым словам -->
                <v-col cols="12" lg="8"
                       order="1" order-lg="4"
                       class="filter__field">
                    <v-text-field v-model="values.searchQuery"
                                  outlined prepend-inner-icon="mdi-magnify"
                                  hide-details label="Поиск по ключевым словам"
                                  @input="onValuesChange"/>
                </v-col>

                <!-- Поиск по месторождению -->
                <v-col v-if="isDesktop"
                       cols="12" lg="4"
                       order="4" order-lg="7"
                       class="filter__field">
                    <location-search v-model="values.locationId"
                                     label="Поиск по месторождению"
                                     :hide-details="true"
                                     @input="onValuesChange">
                        <template v-slot:prepend-inner>
                            <v-icon class="material-icons-outlined">place</v-icon>
                        </template>
                    </location-search>
                </v-col>

                <!-- Радиус поиска месторождений -->
                <v-col cols="12" lg="4"
                       order="5" order-lg="8"
                       class="filter__field">
                    <app-text-field-slider v-model="values.locationRadius"
                                           :max="500"
                                           hide-details
                                           label="Радиус"
                                           postfix="км."
                                           @input="onValuesChange"/>
                </v-col>

                <!-- Чекбоксы -->
                <v-col cols="12" lg="4"
                       order="6" order-lg="3"
                       class="filter__field">
                    <div class="d-flex h-100">
                        <div class="d-flex flex-column justify-center w-100">
                            <v-divider class="d-lg-none mb-4"/>

                            <v-checkbox v-model="values.offersFromLegalOnly"
                                        :true-value="true"
                                        :false-value="null"
                                        label="Только юридические лица и ИП"
                                        hide-details
                                        @change="onValuesChange"/>

                            <v-divider class="d-lg-none mt-4"/>
                        </div>
                    </div>
                </v-col>

                <!-- Очистить фильтр -->
                <v-col v-if="isDesktop"
                       cols="12" lg="4"
                       order="7" order-lg="6"
                       class="filter__btn mt-auto mt-lg-0">
                    <v-btn color="primary" outlined block
                           :disabled="isFilterEmpty"
                           @click="clear">
                        Очистить фильтр
                    </v-btn>
                </v-col>

                <!-- Применить фильтр -->
                <v-col cols="12" lg="4"
                       order="8"
                       class="filter__btn"
                       :class="{ 'fixed-bottom pa-4 blue lighten-5': isMobile }">
                    <v-btn id="btn-304"
                           color="primary" block
                           :loading="loading"
                           @click="apply">
                        Показать {{ countFormat }}
                    </v-btn>
                </v-col>
            </v-row>

            <!-- Мобайл -->
            <template v-if="isMobile">
                <!-- Очистить фильтр -->
                <v-container v-show="!isFilterEmpty"
                             :style="{ top: '56px' }"
                             class="container--dense blue lighten-5 text-small-1 absolute-top">
                    <v-row align="center" justify="space-between">
                        <v-col cols="auto">
                            Выбрано: {{ appliedValuesCount }}
                        </v-col>

                        <v-col cols="auto">
                            <div class="primary--text"
                                 @click="clear">
                                Очистить фильтр
                            </div>
                        </v-col>
                    </v-row>
                </v-container>

                <!-- Диалог выбора категорий -->
                <app-fullscreen-dialog v-model="dialogs.category"
                                       title="Категория"
                                       with-back-btn
                                       eager
                                       @click:back="dialogs.category = false">
                    <category-select v-model="values.arCategoryCode"
                                     url="/api/categories/"
                                     theme="custom"
                                     multiple
                                     :invoke-after-reach="false"
                                     :sub-categories.sync="subCategories"
                                     @input="onValuesChange"
                                     @update:sub-categories="onSubCategoryUpdate">
                        <template v-slot:items="{ onSelect, items }">
                            <v-list style="margin-bottom: 160px">
                                <v-list-item v-for="item in items" :key="item.id"
                                             class="l-border bwb-1">
                                    <v-checkbox v-model="values.arCategoryCode"
                                                :value="item.code"
                                                hide-details
                                                class="w-100"
                                                @change="onSelect">
                                        <template v-slot:label>
                                            <div class="subtitle-1">{{ item.name }}</div>
                                        </template>
                                    </v-checkbox>
                                </v-list-item>
                            </v-list>

                            <v-container class="container--dense white fixed-bottom">
                                <v-row>
                                    <v-col cols="12">
                                        <v-btn color="primary" outlined block
                                               :disabled="isFilterEmpty"
                                               @click="clear">
                                            Очистить фильтр
                                        </v-btn>
                                    </v-col>

                                    <v-col cols="12">
                                        <v-btn color="primary" block
                                               @click="dialogs.category = false">
                                            Выбрать
                                        </v-btn>
                                    </v-col>
                                </v-row>
                            </v-container>
                        </template>
                    </category-select>
                </app-fullscreen-dialog>

                <!-- Диалог выбора подкатегорий -->
                <app-fullscreen-dialog v-model="dialogs.subCategory"
                                       title="Подкатегория"
                                       with-back-btn
                                       @click:back="dialogs.subCategory = false">
                    <v-list style="margin-bottom: 160px">
                        <v-list-item v-for="item in subCategories" :key="item.id"
                                     class="l-border bwb-1">
                            <v-checkbox v-model="values.arSubcategoryId"
                                        :value="item.id"
                                        hide-details
                                        class="w-100"
                                        @change="onValuesChange">
                                <template v-slot:label>
                                    <div class="subtitle-1">{{ item.name }}</div>
                                </template>
                            </v-checkbox>
                        </v-list-item>
                    </v-list>

                    <v-container class="container--dense white fixed-bottom">
                        <v-row>
                            <v-col cols="12">
                                <v-btn color="primary" outlined block
                                       :disabled="isFilterEmpty"
                                       @click="clear">
                                    Очистить фильтр
                                </v-btn>
                            </v-col>

                            <v-col cols="12">
                                <v-btn color="primary" block
                                       @click="dialogs.subCategory = false">
                                    Выбрать
                                </v-btn>
                            </v-col>
                        </v-row>
                    </v-container>
                </app-fullscreen-dialog>

                <!-- Диалог выбора месторождения -->
                <app-fullscreen-dialog v-model="dialogs.location"
                                       title="Месторождение"
                                       with-back-btn
                                       eager
                                       @click:back="dialogs.location = false">
                    <v-container class="container--dense">
                        <location-search v-model="values.locationId"
                                         label="Поиск по месторождению"
                                         :hide-details="true"
                                         @input="onLocationSelect( $event, onValuesChange )">
                            <template v-slot:prepend-inner>
                                <v-icon class="material-icons-outlined">place</v-icon>
                            </template>
                        </location-search>
                    </v-container>

                    <v-container class="container--dense white fixed-bottom">
                        <v-row>
                            <v-col cols="12">
                                <v-btn color="primary" outlined block
                                       :disabled="isFilterEmpty"
                                       @click="clear">
                                    Очистить фильтр
                                </v-btn>
                            </v-col>

                            <v-col cols="12">
                                <v-btn color="primary" block
                                       @click="dialogs.location = false">
                                    Выбрать
                                </v-btn>
                            </v-col>
                        </v-row>
                    </v-container>
                </app-fullscreen-dialog>
            </template>
        </template>
    </app-filter>
</template>

<script>
    import AppSelect           from '~/components/forms/elements/AppSelect';
    import AppFilter           from '~/components/ui/AppFilter';
    import LocationSearch      from '~/components/ui/location/LocationSearch';
    import CategorySelect      from '~/components/ui/category/CategorySelect';
    import AppTextFieldSlider  from '~/components/forms/elements/AppTextFieldSlider';
    import AppFullscreenDialog from '~/components/dialogs/AppFullscreenDialog';

    export default {
        name: 'ExecutorFilter',
        components: { AppFullscreenDialog, AppTextFieldSlider, CategorySelect, LocationSearch, AppFilter, AppSelect },
        data: () => ( {
            values: {
                arCategoryCode: [],
                arSubcategoryId: [],
                searchQuery: undefined,
                offersFromLegalOnly: undefined,
                locationId: undefined,
                locationRadius: undefined,
            },
            description: [
                {
                    code: 'arCategoryCode',
                    type: Array,
                    arrayOf: String,
                    name: 'category',
                },
                {
                    code: 'arSubcategoryId',
                    type: Array,
                    arrayOf: Number,
                    name: 'subcategory',
                },
                {
                    code: 'offersFromLegalOnly',
                    type: Boolean,
                    name: 'offers-from-legal',
                },
                {
                    code: 'locationId',
                    type: Number,
                    name: 'location',
                },
            ],
            subCategories: [],
            countEndings: [ 'исполнителя', 'исполнителей', 'исполнителей' ],
            dialogs: {
                category: false,
                subCategory: false,
                location: false,
            },
        } ),
        methods: {
            onSubCategoryUpdate( value ) {
                if ( !this.values.arSubcategoryId.length ) return;

                const arSubCategoryId = [];

                this.values.arSubcategoryId.forEach( id => {
                    const subCategory = value.find( item => item.id === id );

                    if ( subCategory ) {
                        arSubCategoryId.push( id );
                    }
                } );

                this.values.arSubcategoryId = arSubCategoryId;
            },
            onLocationSelect( value, callback ) {
                if ( Number.isInteger( value ) ) {
                    this.dialogs.location = false;
                }

                callback();
            },
        },
    };
</script>
