<template>
	<div class="dejavu-form">
		<form :id="this.content.attributes.id" @submit="this.validateAndSend"
			:method="this.content.attributes.method" 
			:action="this.content.attributes.action"
			:style="this.content.attributes.style">
			<div v-for="(input, index) in this.content.inputs" :key="input.id">
				<transition name="slide">
					<span v-if="this.errors[index] !== undefined" class="dejavu-error">{{this.errors[index]}}</span>
				</transition>
				<label v-if="input.label" :for="input.name">
					{{input.label}}:{{input.validation && input.validation.required ? '*' : ''}}
				</label>
				<div v-if="input.type === 'select'">
					<select :id="input.attributes && input.attributes.id ? input.attributes.id : ''" 
							:class="input.attributes && input.attributes.class ? input.attributes.class : ''" 
							:style="input.attributes && input.attributes.style ? input.attributes.style : ''" 
							:value="input.value"
							:disabled="input.disabled"
							:required="input.required"
							:placeholder="input.placeholder ? input.placeholder : ''"
							@input="update(index, $event.target.value)">
							<option v-for="option in (input.options ? input.options : [])" 
									:key="option.value ? option.value :''" 
									:value="option.value ? option.value :''" 
									:selected="option.selected">
									{{option.display ? option.display : ''}}
							</option>
					</select>
				</div>
				<div v-else-if="input.type === 'textarea'">
					<textarea :class="input.attributes && input.attributes.class ? input.attributes.class : ''" 
							:style="input.attributes && input.attributes.style ? input.attributes.style : ''" 
							:type="input.type"
							:disabled="input.disabled"
							:placeholder="input.placeholder ? input.placeholder : ''"
							:checked="input.checked ? input.checked : ''"
							:step="input.step ? input.step : ''"
							:row="input.row ? input.row : ''"
							:cols="input.cols ? input.cols : ''"
							:required="input.required"
							:value="input.value"
							@input="update(index, $event.target.value)"/>
				</div>
				<div v-else>
					<input :id="input.attributes && input.attributes.id ? input.attributes.id : ''"
							:ref="input.name"
							:class="`${ ( input.type && input.type == 'checkbox' ) ? 'dejavu-checkbox' : '' }
							${ ( input.type && input.type == 'radio' ) ? 'dejavu-radio' : '' } 
							${ ( input.type && input.type == 'submit' ) ? 'dejavu-submit' : '' }
							${ (input.attributes && input.attributes.class ? input.attributes.class : '') }`" 
							:style="input.attributes && input.attributes.style ? input.attributes.style : ''" 
							:type="input.type"
							:disabled="input.disabled"
							:value="input.value" 
							:placeholder="input.placeholder ? input.placeholder : ''"
							:checked="input.checked ? input.checked : ''"
							:step="input.step ? input.step : ''"
							:multiple="input.multiple"
							:required="input.required"
							@input="update(index, $event.target.value)"
							@change="changed(index)">
							<span v-if="input.type && ( input.type == 'checkbox' || input.type == 'radio' )">
								{{ input.display ? input.display : '' }}
							</span>
				</div>
			</div>
		</form>
	</div>
</template>

<script>
	/* eslint-disable */
	import { defineComponent } from 'vue';

	const REQUIRED_ERROR 	= 0; const REQUIRED_ERROR_MSG 	= "is required";
	const MATCH_ERROR 		= 1; const MATCH_ERROR_MSG 		= "doesn't match regex";
	const MIN_ERROR 		= 2; const MIN_ERROR_MSG 		= "cannot be minor than";
	const MAX_ERROR 		= 3; const MAX_ERROR_MSG 		= "cannot be greater than";
	const EXTENSION_ERROR	= 4; const EXTENSION_ERROR_MSG 	= "extensions not in";

	export default /*#__PURE__*/defineComponent({
		name: 'DejavuForm', // vue component name
		data() {
			return {
				results: this.content,
				errors: {}
			};
		},
		props: {
			content: Object,
		},
		methods: {
			/* update file inputs */
			changed(index) {
				if (this.content.inputs[index].type == "file" || this.content.inputs[index].type == "image") {
					this.content.inputs[index].files = this.$refs[this.content.inputs[index].name].files;
				}
			},
			/* update inputs */
			update(index, value) {
				this.content.inputs[index].value = value;
				this.errors[index] = undefined;
			},
			/* return sum of all files sizes */
			getFileListSize(files) {
				let size = 0;
				for (let i = 0; i < files.length; i++) {
					let file = files.item(i)
					size += file.size;
				}
				return size;
			},
			/* convert string memory format to bytes */
			memoryFormatToBytes(format) {
				return format.toUpperCase().replace(/(\d+)+(\.(\d+))?\s?(k|m|g|t)?b?/i, 
					function(value, p1, p2, p3, p4) { 
						return parseFloat(p1 + (p2 || ""))*({ 'K' : 1<<10, 'M' : 1<<20, 'G' : 1<<30, 'T' : 1<<40 }[p4] || 1); 
					}
				);
			},
            /* check if one file in list hasn't a valid extension */
            hasUnvalidExtensions(files, extensions) {
                for (let i = 0; i < files.length; i++) {
                    let splitted_name = files.item(i).name.split('.');
                    if (!extensions.includes( splitted_name[splitted_name.length - 1] ) ) {
                        return true
                    }
                }
                return false
            },
			/* check if value is empty */
			isEmpty(v) { 
				return ( v === undefined || v === null || v === "" ) 
			}
			,
			/* validate and send form data */
			validateAndSend(event) {
				event.preventDefault(); 

				let invalid_inputs = [];

				function isArray    (v) { return ( typeof v === "array" || v instanceof Array ) }
				function isObject   (v) { return ( typeof v === "object" || v instanceof Object ) }
				function isString   (v) { return ( typeof v === "string" || v instanceof String ) }
				function isNumber   (v) { return ( typeof v === "number" || v instanceof Number ) }
				function isFileList (v) { return ( v instanceof FileList ) }
				function isDate     (v) { return ( v instanceof Date ) }

				function errorIndexToKey(index) {
					if (index === REQUIRED_ERROR) {
						return "required"
					} else if (index === MATCH_ERROR) {
						return "match"
					} else if (index === MIN_ERROR) {
						return "min"
					} else if (index === MAX_ERROR) {
						return "max"
					} else if (index === EXTENSION_ERROR) {
						return "extensions"
					}
					return null
				}

				this.content.inputs.forEach( (input, index) => {
					if (this.content.inputs[index].validation !== undefined) {
						for ( const [rule, rule_value] of Object.entries(this.content.inputs[index].validation) ) {
							let error = null; let message = null;
							let input_value = ( ["file", "image"].includes(this.content.inputs[index].type) ) ? 
							this.content.inputs[index].files : this.content.inputs[index].value;

							/* required check */
							if ( 
								(rule === "required" && rule_value === true) && (
								this.isEmpty(input_value) ||
								( isArray(input_value) && input_value.length == 0 ) || 
								( isObject(input_value) && input_value.length == 0 ) ||
								( isFileList(input_value) && input_value.length == 0 ))
							) {
								error = REQUIRED_ERROR; message = `${input.name} ${REQUIRED_ERROR_MSG}`;
							}

							/* match check */
							if ( rule === "match" && isString(input_value) ) {
								if ( input_value.match(rule_value) === null ) {
									error = MATCH_ERROR; message = `${input.name} ${MATCH_ERROR_MSG} ${rule_value}`;
								}
							}

							/* min error */
							if ( rule === "min" && (
								( isString(input_value) && isDate(rule_value) && Date.parse( new Date(input_value) ) < Date.parse(rule_value) ) ||
								( isString(input_value) && input_value.length < rule_value) || 
								( isNumber(input_value) && input_value < rule_value) ||
								( isFileList(input_value) && ( this.getFileListSize(input_value) < this.memoryFormatToBytes(rule_value) ) )
							) ) {
								error = MIN_ERROR; message = `${input.name} ${MIN_ERROR_MSG} ${rule_value}`;
							}

							/* max error */
							if ( rule === "max" && (
								( isString(input_value) && isDate(rule_value) && Date.parse( new Date(input_value) ) > Date.parse(rule_value) ) ||
								( isString(input_value) && input_value.length > rule_value) || 
								( isNumber(input_value) && input_value > rule_value) ||
								( isFileList(input_value) && ( this.getFileListSize(input_value) > this.memoryFormatToBytes(rule_value) ) )
							) ) {
								error = MAX_ERROR; message = `${input.name} ${MAX_ERROR_MSG} ${rule_value}`;
							}

							/* extension error */
                            if ( rule === "extensions" && isFileList(input_value) && this.hasUnvalidExtensions(input_value, rule_value) ) {
                                error = EXTENSION_ERROR; message = `${input.name} ${EXTENSION_ERROR_MSG} [${rule_value.join(", ")}]`;
                            }

							if ( error !== null ) {
								if ( input.errors !== undefined && isString(input.errors[errorIndexToKey(error)]) ) {
									message = input.errors[errorIndexToKey(error)]
								}
								invalid_inputs.push({ index: index, error: error, message: message })
							}
						}
					}
				})
				invalid_inputs.length > 0 ? this.showErrors(invalid_inputs) : this.sendForm();
			},
			/* display errors on invalid inputs */
			showErrors(invalid_inputs) {
				invalid_inputs.forEach( error => {
					this.errors[error.index] = error.message
				} )
			},
			/* get body from content */
			getContent() {
				if (this.content.header === "application/json") { /**TODO: write JSON version */
				} else {
					let form = new FormData();
					this.content.inputs.forEach( input => {
						if ( input.name && !this.isEmpty(input.value) ) {
							let key = input.name
							let value = input.value
							form.append( key, value);
						}
					})
					return form
				}
			},
			/* send form */
			async sendForm() {
				process = true;
				/* check if parent has method */
				/* if method exists before send await response, if true continue else don't send */
				if (this.$parent.formBefore !== undefined) {
					process = await this.$parent.formBefore();
				}
				if (process === true) {
					let endpoint = this.content.action ? this.content.action : () => {  
						console.warn('DejavuForm doesn\'t have an action setted'); 
						return '#'; 
					}
					let body = this.getContent();
					let request_body = new Object();

					let headers = { 'Content-Type': this.content.type };
					if (this.content.xsrf !== undefined) {
						Object.assign( headers, { [this.content.xsrf.key] : this.content.xsrf.value } );
					}
					Object.assign( request_body, JSON.stringify({ 'headers' : headers }) );
					Object.assign( request_body, { 'method' : this.content.method ? this.content.method : 'GET' } );
					Object.assign( request_body, { 'mode' : this.content.mode ? this.content.mode : 'cors' } );
					Object.assign( request_body, { 'cache' : this.content.cache ? this.content.cache : 'default' } );
					Object.assign( request_body, { 'credentials' : this.content.credentials ? this.content.credentials : 'same-origin' } );
					Object.assign( request_body, { 'redirect' : this.content.redirect ? this.content.redirect : 'follow' } );
					Object.assign( request_body, { 'referrerPolicy' : this.content.referrerPolicy ? this.content.referrerPolicy : 'no-referrer-when-downgrade' } );
					Object.assign( request_body, { 'body' : body } );

					let request = new Request(endpoint, request_body);

					const response = await fetch(request)
					this.$parent.formCallback( response );
				}
			}
		}
	});
</script>

<style scoped>
	/* transictions */
	.slide-enter-active, .slide-leave-active {
		transition: all 0.5s;
	}
	.slide-enter-to, .slide-leave-from {
		position: relative;
		top: 0;
		opacity: 100%;
	}
	.slide-enter-from, .slide-leave-to {
		position: relative;
		top: -2em;
		opacity: 0%;
	}
	/* style */
	label {
		min-width: 100%;
		display: block;
	}
	div {
		padding-top: 0.5em;
		padding-bottom: 0.5em;
	}
	.dejavu-form form {
		padding: 1em;
	}
	input, select, textarea {
		padding: 0.2em;
		width: 100%;
		border-radius: 8px;
		border-width: 0;
		background: #242838;
		color: white;
	}
	.dejavu-submit {
		transition: all 0.1s ease-in;
		padding: 0.8em;
		background:#669900;
	}
	.dejavu-submit:hover {
		background: #80bf02;
	}
	.dejavu-submit:active {
		background: #507500;
	}
	.dejavu-form {
		color: white;
		display: block;
		width: 100%;
		background: #292d3e;
	}
	.dejavu-form p {
		margin: 0 0 1em;
	}
	.dejavu-checkbox, .dejavu-radio {
		width: auto;
	}
	.dejavu-error {
		color: red;
		margin-bottom: 1em;
		display: block;
	}
</style>
