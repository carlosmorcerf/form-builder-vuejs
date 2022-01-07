<template>
	<div v-bind:style="containerStyles" v-bind:id="formId">
		<div v-for="(field, index) in fields" class="my-3 p-2 bg-white w-100 field-row" :key="field.id" v-bind:drag-index="index" v-on:dragover="dragover($event, 'field-row')" v-on:drop="drop($event, 'field-row', fieldMove)" v-on:dragenter="dragenter($event, 'field-row')" v-on:dragleave="dragleave($event, 'field-row')">
			<div v-if="field.isOpen === false" class="p-3 border rounded shadow-sm w-100 d-flex justify-content-between align-items-center">
				<div>
					<p class="m-0">
						<b>{{ field.component.label }} ({{ field.component.type }}) <span v-if="field.component.validations.required === true">*</span></b>
					</p>
					<label v-if="!field.isValid" class="text-danger"> * Campo inválido </label>
				</div>
				<div class="d-flex justify-content-end align-items-center">
					<div class="custom-control custom-switch">
						<input type="checkbox" class="custom-control-input" v-bind:id="'field_' + field.id" v-model="field.component.status" />
						<label class="custom-control-label" v-bind:for="'field_' + field.id"></label>
					</div>
					<a href="javascript:;" class="btn btn-sm btn-primary mr-1" v-on:click="field.open()">
						<i class="fa fa-pencil"></i>
					</a>
					<a href="javascript:;" class="btn btn-sm btn-danger mr-1" v-on:click="removeField(index)">
						<i class="fa fa-trash-o"></i>
					</a>
					<a href="javascript:;" class="btn" style="cursor: move" draggable="true" v-on:dragstart="dragstart($event, 'field-row')" v-on:dragend="dragend()">
						<i class="fa fa-ellipsis-v"></i>
					</a>
				</div>
			</div>

			<div v-if="field.isOpen === true" class="card my-1" style="border-style: dotted; border-color: #333">
				<div class="card-header">
					<b>{{ field.component.label }}</b>
				</div>
				<div class="card-body">
					<div class="alert alert-danger" v-if="field.isValid === false">
						<p v-for="(error, eIndex) in field.errors" :key="eIndex" class="m-0">
							{{ error }}
						</p>
					</div>

					<div class="form-group row">
						<label class="col-12 col-md-4 col-form-label col-form-label-sm">Label</label>
						<div class="col-12 col-md-8">
							<input v-model="field.component.label" type="text" class="form-control form-control-sm" />
						</div>
						<small class="form-text text-muted"></small>
					</div>

					<div class="form-group row">
						<label class="col-12 col-md-4 col-form-label col-form-label-sm">Tipo do campo</label>
						<div class="col-12 col-md-8">
							<select v-model="field.component.type" class="form-control form-control-sm">
								<option value="label">Label</option>
								<option value="text">Texto</option>
								<option value="textarea">Textarea</option>
								<option value="email">E-mail</option>
								<option value="password">Senha</option>
								<option value="phone">Telefone</option>
								<option value="date">Data</option>
								<option value="cpf">CPF</option>
								<option value="cnpj">CNPJ</option>
								<option value="cep">CEP</option>
								<option value="select">Select</option>
								<option value="checkbox">Checkbox</option>
								<option value="radio">Radio</option>
							</select>
						</div>
					</div>

					<div v-if="field.component.type && field.component.type != 'label'" class="p-2">
						<hr />

						<ul class="nav nav-tabs">
							<li class="nav-item">
								<a class="nav-link" v-bind:class="field.tab == 'config' ? 'active' : ''" v-on:click="field.setTab('config')" href="javascript:;">Configurações</a>
							</li>
							<li class="nav-item" v-if="field.haveOptions()">
								<a class="nav-link" v-bind:class="field.tab == 'options' ? 'active' : ''" v-on:click="field.setTab('options')" href="javascript:;">Opções</a>
							</li>
							<li class="nav-item">
								<a class="nav-link" v-bind:class="field.tab == 'validation' ? 'active' : ''" v-on:click="field.setTab('validation')" href="javascript:;">Validação</a>
							</li>
						</ul>

						<div v-if="field.tab == 'config'" class="mt-3">
							<div class="form-group row" v-if="field.isTextBox()">
								<label class="col-12 col-md-4 col-form-label col-form-label-sm">Placeholder</label>
								<div class="col-12 col-md-8">
									<input v-model="field.component.placeholder" type="text" class="form-control form-control-sm" />
								</div>
								<small class="form-text text-muted"></small>
							</div>

							<div class="form-group row">
								<label class="col-12 col-md-4 col-form-label col-form-label-sm">Texto de ajuda</label>
								<div class="col-12 col-md-8">
									<input v-model="field.component.tip" type="text" class="form-control form-control-sm" />
								</div>
								<small class="form-text text-muted"></small>
							</div>

							<div class="form-group row">
								<label class="col-12 col-md-4 col-form-label col-form-label-sm">Valor inicial</label>
								<div class="col-12 col-md-8">
									<input v-model="field.component.defaultValue" type="text" class="form-control form-control-sm" />
								</div>
							</div>

							<div class="form-group row">
								<label class="col-12 col-md-4 col-form-label col-form-label-sm">CSS Class</label>
								<div class="col-12 col-md-8">
									<input v-model="field.component.cssClasses" type="text" class="form-control form-control-sm" />
								</div>
								<small class="form-text text-muted"></small>
							</div>

							<div class="form-group row">
								<label class="col-12 col-md-4 col-form-label col-form-label-sm">Identificador único</label>
								<div class="col-12 col-md-8">
									<input v-model="field.component.name" readonly type="text" class="form-control form-control-sm" />
								</div>
							</div>
						</div>

						<div v-if="field.tab == 'options' && field.haveOptions()" class="mt-3">
							<div
								v-for="(option, optionIndex) in field.component.values"
								class="my-3 p-2 bg-white w-100 field-option-row d-flex align-items-center"
								:key="optionIndex"
								v-bind:drag-index="optionIndex"
								v-on:dragover="dragover($event, 'field-option-row')"
								v-on:drop="
									drop($event, 'field-option-row', (a, b) => {
										field.moveValueOption(a, b);
									})
								"
								v-on:dragenter="dragenter($event, 'field-option-row')"
								v-on:dragleave="dragleave($event, 'field-option-row')"
							>
								<div class="row align-items-center mr-2"><label class="col-auto">Label: </label> <input type="text" v-model="option.label" class="form-control col" placeholder="Label" /></div>
								<div class="row align-items-center mr-2"><label class="col-auto">Valor:</label> <input type="text" v-model="option.value" class="form-control col" placeholder="Valor" /></div>
								<div>
									<a href="javascript:;" class="btn btn-sm btn-danger mr-1" v-on:click="field.removeValueOption(optionIndex)">
										<i class="fa fa-trash-o"></i>
									</a>
								</div>
								<div>
									<a href="javascript:;" class="btn" style="cursor: move" draggable="true" v-on:dragstart="dragstart($event, 'field-option-row')" v-on:dragend="dragend()">
										<i class="fa fa-ellipsis-v"></i>
									</a>
								</div>
							</div>
							<div style="border: 1px dotted #000" class="p-2 text-center">
								<a class="btn btn-primary btn-sm" href="javascript:;" v-on:click="field.addOption()">+ Adicionar opção</a>
							</div>
						</div>

						<div v-if="field.tab == 'validation'" class="mt-3">
							<div class="form-group row">
								<div class="col-12 col-md-8 offset-md-4">
									<div class="form-check">
										<input v-model="field.component.validations.required" v-bind:id="'required_' + field.id" class="form-check-input" type="checkbox" />
										<label v-bind:for="'required_' + field.id" class="form-check-label">Obrigatório</label>
									</div>
								</div>
							</div>
							<div class="form-group row">
								<label class="col-12 col-md-4 col-form-label col-form-label-sm">Mínimo de characters</label>
								<div class="col-12 col-md-8">
									<input v-model="field.component.validations.minlength" type="text" class="form-control form-control-sm" />
								</div>
							</div>
							<div class="form-group row">
								<label class="col-12 col-md-4 col-form-label col-form-label-sm">Máximo de characters</label>
								<div class="col-12 col-md-8">
									<input v-model="field.component.validations.maxlength" type="text" class="form-control form-control-sm" />
								</div>
							</div>
							<div class="form-group row">
								<label class="col-12 col-md-4 col-form-label col-form-label-sm">Expressão regular (regex)</label>
								<div class="col-12 col-md-8">
									<input v-model="field.component.validations.regexPattern" type="text" class="form-control form-control-sm" />
								</div>
							</div>
						</div>
					</div>
				</div>
				<div class="card-footer text-center">
					<a href="javascript:;" v-on:click="removeField(index)" class="btn btn-sm btn-danger mr-2">
						<i class="fa fa-trash-o"></i>
					</a>
					<a href="javascript:;" v-if="field.recent === false" v-on:click="field.cancelChanges()" class="btn btn-secondary btn-sm mr-2">
						<i class="fa fa-undo"></i>
					</a>
					<a href="javascript:;" v-on:click="field.saveChanges()" class="btn btn-primary btn-sm">
						<i class="fa fa-floppy-o"></i>
					</a>
				</div>
			</div>
		</div>

		<div style="border: 1px dotted #000" class="p-3 text-center">
			<a class="btn btn-sm btn-outline-primary" href="javascript:;" v-on:click="addField()">
				<i class="fa fa-plus"></i>
				Novo campo
			</a>
		</div>

		<div class="card mt-3" v-if="configuringForm === true">
			<div class="card-header">
				<b>Configurações do formulário</b>
			</div>

			<div class="card-body">
				<div class="form-group row">
					<label class="col-12 col-md-4 col-form-label col-form-label-sm">CSS Class (form)</label>
					<div class="col-12 col-md-8">
						<input v-model="formConfig.cssClasses" type="text" class="form-control form-control-sm" />
					</div>
					<small class="form-text text-muted">Estas classes serão incluidas na tag &lt;form&gt;</small>
				</div>
				<div class="form-group row">
					<label class="col-12 col-md-4 col-form-label col-form-label-sm">CSS Class (campos)</label>
					<div class="col-12 col-md-8">
						<input v-model="formConfig.fieldsCssClasses" type="text" class="form-control form-control-sm" />
					</div>
					<small class="form-text text-muted">Estas classes serão incluídas em todos os campos</small>
				</div>

				<div class="form-group row">
					<label class="col-12 col-md-4 col-form-label col-form-label-sm">Texto botão enviar (submit)</label>
					<div class="col-12 col-md-8">
						<input v-model="formConfig.submitBtnText" type="text" class="form-control form-control-sm" />
					</div>
				</div>

				<div class="form-group row">
					<label class="col-12 col-md-4 col-form-label col-form-label-sm">CSS Class (submit)</label>
					<div class="col-12 col-md-8">
						<input v-model="formConfig.submitBtnCssClasses" type="text" class="form-control form-control-sm" />
					</div>
				</div>

				<div class="form-group row">
					<div class="col-12 col-md-8 offset-md-4">
						<div class="form-check">
							<input v-model="formConfig.resetBtnStatus" id="resetBtnStatus" class="form-check-input" type="checkbox" />
							<label for="resetBtnStatus" class="form-check-label">Mostrar botão cancelar? (reset)</label>
						</div>
					</div>
				</div>

				<div class="form-group row">
					<label class="col-12 col-md-4 col-form-label col-form-label-sm">Texto botão cancelar (reset)</label>
					<div class="col-12 col-md-8">
						<input v-model="formConfig.resetBtnText" type="text" class="form-control form-control-sm" />
					</div>
				</div>
				<div class="form-group row">
					<label class="col-12 col-md-4 col-form-label col-form-label-sm">CSS Class (reset)</label>
					<div class="col-12 col-md-8">
						<input v-model="formConfig.resetBtnCssClasses" type="text" class="form-control form-control-sm" />
					</div>
				</div>
			</div>

			<div class="card-footer text-center">
				<a href="javascript:;" v-on:click="cancelFormConfig()" class="btn btn-secondary btn-sm mr-2">
					<i class="fa fa-undo"></i>
				</a>
				<a href="javascript:;" v-on:click="saveFormConfig()" class="btn btn-primary btn-sm">
					<i class="fa fa-floppy-o"></i>
				</a>
			</div>
		</div>

		<div class="mt-3">
			<span v-if="formErrorMessage" class="text-danger">{{ formErrorMessage }}</span>
		</div>

		<div style="my-2" class="p-2 d-flex justify-content-end align-items-center">
			<a class="btn btn-secondary mr-2" href="javascript:;" v-on:click="openFormConfig()">
				<i class="fa fa-cog"></i>
				Configurar
			</a>
			<a class="btn btn-secondary mr-2" href="javascript:;" v-on:click="cancelFormChanges()">
				<i class="fa fa-undo"></i>
				Cancelar
			</a>
			<a class="btn btn-primary" href="javascript:;" v-on:click="saveForm()">
				<i class="fa fa-floppy-o"></i>
				Salvar
			</a>
		</div>
	</div>
</template>

<style lang="css" scoped>
.dragging {
	opacity: 0.5;
}
.dragover {
	border: 2px dashed rgb(233, 53, 53) !important;
}
</style>

<script>
var $ = window.jQuery;

function makeRandomId(length) {
	let result = "";
	const characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz_";
	for (let i = 0; i < length; i++) {
		result += characters.charAt(Math.floor(Math.random() * characters.length));
	}
	return result;
}

var factory = {
	field: function (component = null) {
		var opened = false;
		var recent = false;
		if (!component) {
			component = {
				type: null,
				label: "Novo campo",
				name: makeRandomId(14),
				placeholder: "",
				defaultValue: "",
				tip: "",
				status: true,
				values: [], //{ label: "", value: "" }
				cssClasses: null,
				validations: {
					required: false,
					regexPattern: null,
					minlength: null,
					maxlength: null,
				},
			};
			opened = true;
			recent = true;
		}

		var field = {
			isTextBox: function () {
				if (this.component.type == "text" || this.component.type == "textarea" || this.component.type == "password" || this.component.type == "email" || this.component.type == "phone" || this.component.type == "date" || this.component.type == "cpf" || this.component.type == "cnpj" || this.component.type == "cep") {
					return true;
				}
				return false;
			},
			haveOptions: function () {
				if (this.component.type == "select" || this.component.type == "radio" || this.component.type == "checkbox") {
					return true;
				}
				return false;
			},
			recent: recent,
			oldValue: null,
			isValid: null,
			isOpen: false,
			errors: [],
			tab: "config",
			id: "id" + makeRandomId(14),
			addOption: function () {
				this.component.values.push({ label: "", value: "" });
			},
			cancelChanges: function () {
				if (this.oldValue !== null) {
					this.component = JSON.parse(this.oldValue);
				}
			},
			saveChanges: function () {
				if (this.validate() === true) {
					this.oldValue = null;
					this.setTab("config");
					this.isOpen = false;
				}
			},
			open: function () {
				if (!this.isOpen) {
					this.oldValue = JSON.stringify(this.component);
					this.setTab("config");
					this.isOpen = true;
				}
			},
			moveValueOption: function (old_index, new_index) {
				if (new_index >= this.component.values.length) {
					var k = new_index - this.component.values.length + 1;
					while (k--) {
						this.component.values.push(undefined);
					}
				}
				this.component.values.splice(new_index, 0, this.component.values.splice(old_index, 1)[0]);
			},
			removeValueOption: function (index) {
				if (index > -1) {
					this.component.values.splice(index, 1);
				}
			},
			setTab: function (name) {
				this.tab = name;
			},
			validate: function () {
				this.errors = [];

				var letras_underline = /^[A-Za-z_]+$/g;

				if (!this.component.name) {
					this.errors.push("Nome é obrigatorio");
				}
				if (!letras_underline.test(this.component.name)) {
					this.errors.push("Nome pode ter apenas letras e _");
				}
				if (!this.component.label) {
					this.errors.push("Label é obrigatorio");
				}

				if (!this.component.type) {
					this.errors.push("Tipo é obrigatório");
				}

				if (this.haveOptions()) {
					if (this.component.values.length < 1) {
						this.errors.push("Adicione ao menos uma opção");
					} else {
						for (var i = 0; i < this.component.values.length; i++) {
							var option = this.component.values[i];
							if (!option.label) {
								this.errors.push("Label da opção é obrigatório");
							}
							if (!option.value) {
								this.errors.push("Valor da opção " + option.label + " é obrigatório");
							}
						}
					}
				} else {
					this.component.values = [];
				}

				this.isValid = this.errors.length === 0;

				return this.isValid;
			},
			component: component,
		};

		if (opened === true) {
			field.open();
		} else {
			field.validate();
		}

		return field;
	},
	formConfig: {
		cssClasses: null,
		fieldsCssClasses: null,
		submitBtnText: "Enviar",
		submitBtnCssClasses: null,
		resetBtnText: "Cancelar",
		resetBtnCssClasses: null,
		resetBtnStatus: false,
	},
};

export default {
	name: "FormBuilder",
	props: {
		formId: String,
		containerStyles: String,
		initialSchema: Object,
		onSave: Function,
		onCancel: Function,
	},
	data: function () {
		return {
			fields: [],
			formErrorMessage: null,
			lastSchema: null,
			configuringForm: false,
			oldFormConfig: null,
			formConfig: null,
		};
	},
	created: function () {
		if (this.initialSchema) {
			this.applySchema(this.initialSchema);
		}
	},
	methods: {
		openFormConfig: function () {
			if (this.configuringForm) {
				return;
			}

			this.oldFormConfig = JSON.parse(JSON.stringify(this.formConfig));
			this.configuringForm = true;
		},
		saveFormConfig: function () {
			if (!this.configuringForm) {
				return;
			}

			this.configuringForm = false;
		},
		cancelFormConfig: function () {
			if (!this.configuringForm) {
				return;
			}

			this.formConfig = JSON.parse(JSON.stringify(this.oldFormConfig));
		},

		cancelFormChanges: function () {
			this.configuringForm = false;
			this.applySchema(this.lastSchema);

			if (this.onCancel){
				this.onCancel();
			}
		},
		applySchema: function (newSchema) {
			var schema = JSON.parse(JSON.stringify(newSchema));
			if (schema.schemaVersion != "formbuilder-v1") {
				alert("Invalid schema");
			} else {
				this.lastSchema = schema;
				this.formConfig = schema.formConfig ? schema.formConfig : factory.formConfig;
				this.fields = [];
				this.config = {};
				for (var i = 0; i < schema.fields.length; i++) {
					this.addField(schema.fields[i]);
				}
			}
		},
		saveForm: function () {
			var isValid = true;
			var errorMessage = "* Existem erros no formulário!";
			var schema = {
				schemaVersion: "formbuilder-v1",
				fields: [],
			};

			if (this.configuringForm) {
				isValid = false;
				errorMessage = "* Finalize a configuração do formulário para salvar!";
			} else {
				schema.formConfig = this.formConfig;
			}

			for (var i = 0; i < this.fields.length; i++) {
				if (this.fields[i].isOpen !== false) {
					isValid = false;
					errorMessage = "* Existem campos não salvos!";
				}

				if (this.fields[i].validate() !== true) {
					isValid = false;
				} else {
					schema.fields.push(this.fields[i].component);
				}
			}

			if (isValid) {
				errorMessage = null;
				schema = JSON.parse(JSON.stringify(schema));
				if (this.onSave) {
					this.onSave(true, schema, null);
					this.lastSchema = schema;
				}
			} else {
				if (this.onSave) {
					this.onSave(false, null, errorMessage);
				}
			}

			this.formErrorMessage = errorMessage;
		},
		fieldMove: function (old_index, new_index) {
			if (new_index >= this.fields.length) {
				var k = new_index - this.fields.length + 1;
				while (k--) {
					this.fields.push(undefined);
				}
			}
			this.fields.splice(new_index, 0, this.fields.splice(old_index, 1)[0]);
		},
		removeField: function (index) {
			if (index > -1) {
				this.fields.splice(index, 1);
			}
		},
		addField: function (component = null) {
			this.fields.push(factory.field(component));
		},

		// drag drop functions
		drop: function (e, draggable, onDrop) {
			e.stopPropagation();

			var dragging = $(".dragging." + draggable);
			if (dragging.hasClass(draggable)) {
				var dragover = $(".dragover." + draggable);
				if (dragover) {
					var oldIndex = dragging.attr("drag-index");
					var index = dragover.attr("drag-index");
					onDrop(oldIndex, index);
					//this.fieldMove(oldIndex, index);
				}
			}

			$(".dragging." + draggable).removeClass("dragging");
			$(".dragover." + draggable).removeClass("dragover");

			return false;
		},
		dragover: function (e, draggable) {
			e.preventDefault();

			var $el = $(e.target).closest("." + draggable + ":not(.dragging)");
			if ($el.length == 0) return;

			var dragging = $(".dragging." + draggable);
			if (dragging.hasClass(draggable)) {
				$el.addClass("dragover");
			}

			return false;
		},
		dragenter: function (e, draggable) {
			var $el = $(e.target);
			if (!$el.hasClass(draggable) || $el.hasClass("dragging")) {
				return;
			}
			if ($el.length == 0) return;

			var dragging = $(".dragging." + draggable);
			if (dragging.hasClass(draggable)) {
				$el.addClass("dragover");
			}
		},
		dragleave: function (e, draggable) {
			var $el = $(e.target);
			if (!$el.hasClass(draggable) || $el.hasClass("dragging")) {
				return;
			}
			if ($el.length == 0) return;

			var dragging = $(".dragging");
			if (dragging.hasClass(draggable)) {
				$el.removeClass("dragover");
			}
		},
		dragstart: function (e, draggable) {
			$(e.target)
				.closest("." + draggable)
				.addClass("dragging");
		},
		dragend: function () {
			$(".dragging").removeClass("dragging");
			$(".dragover").removeClass("dragover");
		},
	},
};
</script>
