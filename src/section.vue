<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
  <div class="section-container">
    <!-- wwManager:start -->
    <wwSectionEditMenu :sectionCtrl="sectionCtrl"></wwSectionEditMenu>
    <!-- wwManager:end -->

    <!-- This is the background of the section -->
    <wwObject class="background"
              :ww-object="section.data.background"
              ww-category="background"></wwObject>

    <div class="testimonial-section-content">
      <ul class="testimonial-list">
        <li class="testimonial-item-wrapper"
            :class="{selected: isTestimonialSelected(testimonial)}"
            v-for="(testimonial, index) in section.data.testimonials"
            :key="testimonial.uniqueId">
          <!-- wwManager:start -->
          <wwContextMenu
                  class="ww-orange-button"
                  tag="div"
                  :options="elemOptions"
                  v-if="editMode"
                  @remove="removeTestimonial(testimonial)"
                  @addBefore="addTestimonialBefore(index)"
                  @addAfter="addTestimonialAfter(index)">
            <wwOrangeButton></wwOrangeButton>
          </wwContextMenu>
          <!-- wwManager:end -->
          <div class="testimonial-item">
            <div class="testimonial-text">
              <wwObject tag="span"
                        :ww-object="testimonial.text"
              ></wwObject>
            </div>
            <div class="testimonial-content">
              <wwObject tag="span"
                        class="testimonial-logo"
                        :ww-object="testimonial.logo"
              ></wwObject>
              <div class="testimonial-role">
                <wwLayoutColumn tag="div"
                                ww-default="ww-text"
                                :ww-list="testimonial.roleContent"
                                @ww-add="add(testimonial.roleContent, $event)"
                                @ww-remove="remove(testimonial.roleContent, $event)">
                  <wwObject tag="div"
                            ww-default="ww-text"
                            v-for="item in testimonial.roleContent"
                            :key="item.uniqueId"
                            :ww-object="item"
                  >
                  </wwObject>
                </wwLayoutColumn>
              </div>
            </div>
          </div>
        </li>
      </ul>
      <ul class="testimonial-navigator">
        <li v-for="(testimonial) in section.data.testimonials"
            :key="testimonial.uniqueId"
            @click="toggleTestimonial(testimonial)"
        >
          <svg class="testimonial-navigator-item"
               :class="{selected:isTestimonialSelected(testimonial)}"
               width="24"
               height="12"
               viewBox="0 0 24 12"
               fill="none"
               xmlns="http://www.w3.org/2000/svg">
            <rect width="12"
                  height="12"
                  rx="4"
                  fill="#FDC806"/>
          </svg>
        </li>
      </ul>
    </div>
  </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->

<script>

    const wwo = window.wwLib.wwObject;
    const wwu = window.wwLib.wwUtils;

    const LOOP_DURATION = 5000;

    export default {
        name: '__COMPONENT_NAME__',
        props: {
            sectionCtrl: Object
        },
        data: () => ({
            selectedTestimonial: {},
            animation: {
                id: 0,
                startTime: 0,
                duration: 5000
            },
            elemOptions: {
                items: [
                    {
                        text: {
                            en: 'Before',
                            fr: 'Avant'
                        },
                        icon: 'wwi wwi-add',
                        action: 'addBefore'
                    },
                    {
                        text: {
                            en: 'After',
                            fr: 'Apres'
                        },
                        icon: 'wwi wwi-add',
                        action: 'addAfter'
                    },
                    {
                        text: {
                            en: 'Delete',
                            fr: 'Supprimer'
                        },
                        icon: 'wwi wwi-delete',
                        action: 'remove'
                    }
                ]
            }
        }),
        computed: {
            section () {
                return this.sectionCtrl.get();
            },
            editMode () {
                return this.sectionCtrl.getEditMode() === 'CONTENT';
            }
        },
        watch: {
            editMode (next, current) {
                current && this.playLoop();
                next && this.stopLoop();
            }
        },
        created () {
            this.init();
        },
        mounted () {
            if (this.section.data.testimonials.length > 0) {
                this.selectedTestimonial = this.section.data.testimonials[0];
            }
            if (!this.editMode) {
                this.playLoop();
            }
        },
        methods: {
            init () {
                let needUpdate = false;
                this.section.data = this.section.data || {};

                if (!this.section.data.background) {
                    this.section.data.background = wwo.getDefault({
                        type: 'ww-image'
                    });
                    needUpdate = true;
                }

                if (!this.section.data.testimonials) {
                    this.section.data.testimonials = [this.createTestimonial()];
                    needUpdate = true;
                }
                needUpdate && this.update();
            },
            playLoop () {
                const { testimonials } = this.section.data;
                this.animation.id = requestAnimationFrame(() => {
                    const delta = new Date() - this.animation.startTime;
                    if (delta >= LOOP_DURATION) {
                        const length = testimonials.length;
                        const nextIdx = testimonials.indexOf(this.selectedTestimonial) + 1;
                        this.selectedTestimonial = this.section.data.testimonials[nextIdx >= length ? 0 : nextIdx];
                        this.animation.startTime = new Date();
                    }
                    this.playLoop();
                });
            },
            stopLoop () {
                cancelAnimationFrame(this.animation.id);
                this.animation = {
                    ...this.animation,
                    id: 0,
                    startTime: 0
                };
            },
            getNewTestimonial: () => ({
                uniqueId: wwu.getUniqueId(),
                text: wwo.getDefault({ type: 'ww-text' }),
                logo: wwo.getDefault({ type: 'ww-image' }),
                roleContent: []
            }),
            createTestimonial () {
                let testimonial = {};
                if (Array.isArray(this.section.data.testimonials) && this.section.data.testimonials.length > 0) {
                    testimonial = JSON.parse(JSON.stringify(this.section.data.testimonials[0]));
                    wwu.changeUniqueIds(testimonial);
                    testimonial.uniqueId = wwu.getUniqueId();
                    return testimonial;
                }
                return this.getNewTestimonial();
            },
            isTestimonialSelected (testimonial) {
                return testimonial.uniqueId === this.selectedTestimonial.uniqueId;
            },
            toggleTestimonial (testimonial) {
                this.stopLoop();
                this.selectedTestimonial = testimonial;
                if (this.editMode) return;
                this.restartLoop();
            },
            restartLoop () {
                this.$forceUpdate();
                const timer = setTimeout(() => {
                    this.playLoop();
                    clearTimeout(timer);
                }, LOOP_DURATION);
            },
            addTestimonialBefore (index) {
                this.addTestimonialAt(index === 0 ? 0 : index - 1);
                this.update();
            },
            addTestimonialAfter (index) {
                this.addTestimonialAt(index + 1);
                this.update();
            },
            addTestimonialAt (index) {
                const { testimonials } = this.section.data;
                const head = testimonials.slice(0, index);
                const tail = index === 0 ? testimonials : testimonials.slice(index);
                this.section.data.testimonials = [...head, this.createTestimonial(), ...tail];
            }
            ,
            removeTestimonial (testimonial) {
                const { testimonials } = this.section.data;
                if (testimonials.length === 1) return;
                this.section.data.testimonials = testimonials.filter(aTestimonial => aTestimonial !== testimonial);
                this.update();
            },
            update () {
                this.sectionCtrl.update(this.section);
            },

            // --------- EDITOR FUNCTIONS ---------
            // All the codes between /* wwManager:start */ and /* wwManager:end */ are only for editor purposes
            // So It won't in the published website!
            /* wwManager:start */
            add (list, options) {
                try {
                    list.splice(options.index, 0, options.wwObject);
                    this.sectionCtrl.update(this.section);
                } catch (error) {
                    wwLib.wwLog.error('ERROR : ', error);
                }
            },
            remove (list, options) {
                try {
                    list.splice(options.index, 1);
                    this.sectionCtrl.update(this.section);
                } catch (error) {
                    wwLib.wwLog.error('ERROR : ', error);
                }
            }
            /* wwManager:end */
        }
    }
    ;
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang="scss"
       scoped>

  .background {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
  }

  .testimonial-section-content {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    height: 734px;
    margin: auto;
    padding: 100px 24px 0 24px;
    font-family: Work Sans;
    font-style: normal;
    font-weight: 500;
    color: var(--color-true-black);

    @media (min-width: 768px) {
      width: 768px;
      padding: 192px 32px 0 32px;
    }
    @media (min-width: 1024px) {
      width: 768px;
    }
  }

  .testimonial {
    &-list {
      position: relative;
      width: 100%;
      height: auto;
      padding: 0;
      margin: 0;
      min-height: 200px;
      list-style-type: none;

      @media (min-width: 768px) {
        min-height: 150px;
      }

      @media (min-width: 1024px) {
        min-height: 200px;
      }
    }

    &-item-wrapper {
      position: absolute;
      top: 0;
      width: 100%;
      opacity: 0;
      z-index: 0;

      &.selected {
        opacity: 1;
        z-index: 1;
        transition: opacity 2s ease-in-out;
      }
    }

    &-item {
      width: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    &-text {
      height: 80px;
      font-size: 24px;
      line-height: 1;
      text-align: center;
      margin-bottom: 64px;

      @media (min-width: 768px) {
        margin-bottom: 40px;
        height: 100px;
        font-size: 24px;
        line-height: 28px;
      }

      @media (min-width: 1024px) {
        height: 150px;
        font-size: 36px;
        line-height: 42px;
      }
    }

    &-icon {
      margin-bottom: 16px;
    }

    &-content {
      display: flex;
      flex-direction: row;
      justify-items: center;
      margin: auto;
    }

    &-logo {
      width: 64px;
      height: 64px;
      margin-right: 24px;
    }

    &-role {
      display: flex;
      flex-direction: column;
      justify-items: center;
      font-size: 18px;
      line-height: 21px;
    }

    &-navigator {
      display: flex;
      flex-direction: row;
      align-items: center;
      margin: 32px auto 0 auto;
      padding: 0;
      list-style-type: none;
      cursor: pointer;

      @media (min-width: 768px) {
        margin: 64px auto 0 auto;
      }

      @media (min-width: 1024px) {
        margin: 72px auto 0 auto;
      }

      &-item {
        --color-yellow: #FDC806;

        width: 24px;
        height: 12px;
        margin-right: 12px;
        margin-top: 8px;
        pointer-events: all;
        transition: all 1000ms;

        rect {
          transition: all 1000ms;
          fill: var(--color-true-black);
          width: 24px;
          height: 4px;
          rx: 0;
        }

        &.selected {
          margin-right: 0;
          margin-top: 0;

          rect {
            fill: var(--color-yellow);;
            width: 12px;
            height: 12px;
            rx: 4px;
          }
        }
      }
    }

  }

</style>
