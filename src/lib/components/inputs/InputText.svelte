<script>
	import Tooltip from "$lib/components/Tooltip.svelte";
	import { L } from "$lib/js/client/localization/localization.translations.data.client";
	import { getStore } from "$lib/js/client/util.client";
	import { areEqualStrings } from "$lib/js/client/util.inputs.client";
	import { waitFor } from "$lib/js/common/util.common";
	import { onMount } from "svelte";

	export let wrapperClasses,
		inputClasses,
		name,
		type,
		inputmode,
		title,
		value,
		tText,
		validate,
		format,
		valid,
		command,
		disable,
		optional,
		input,
		validInVisual,
		enterKeyHint;

	let focus,
		exitedOnce,
		tVisible,
		editMode,
		firstValue,
		empty = !value;

	const bigScreen = getStore("bigScreen"),
		lang = getStore("lang");

	/* */

	$: onCommand(command);

	onMount(_onMount);

	if (value) {
		editMode = true;
		firstValue = value;
		validInVisual = true;
		valid = "notEdited";
	}

	if (optional || disable) valid = true;

	/* */

	function onCommand() {
		if (!command) return;

		switch (command.type) {
			case "checkValidation":
				if (!exitedOnce) return;

				const validation = validate(value);
				valid = validation;
				tVisible = !validation;
				validInVisual = validation;

				break;
		}

		command = undefined;
	}

	function _onMount() {
		if (input === document.activeElement) focus = true;
	}

	/* */

	async function onFocus(event) {
		if (type !== "email" && type !== "number")
			event.target.setSelectionRange(event.target.value.length, event.target.value.length);

		focus = true;
		tVisible = true;

		if ($bigScreen) return;

		await waitFor(250);
		event.target.scrollIntoView({ inline: "center", block: "center", behavior: "smooth" });
	}

	function onBeforeInput(event) {
		if (!format || !event.data) return;

		const formatted = format(event.target.value + event.data, event.data);
		if (!formatted) return;

		const newValue = formatted.newValue;
		if (newValue) {
			event.target.value = newValue;
			event.preventDefault();

			return;
		}

		if (formatted.preventDefault) event.preventDefault();
	}

	function onInput(event) {
		const newValue = (value = event.target.value);

		if (!newValue) {
			empty = true;

			if (optional) {
				valid = true;
				validInVisual = undefined;
				tVisible = exitedOnce = false;

				return;
			}
		} else empty = false;

		const validation = validate(newValue, name);

		if (editMode && areEqualStrings(newValue, firstValue)) valid = "notEdited";
		else valid = validation;

		tVisible = !validation;

		if (exitedOnce) validInVisual = validation;
	}

	function onBlur(event) {
		focus = false;

		if (optional && !event.target.value) {
			tVisible = false;

			return;
		}

		if (exitedOnce) {
			tVisible = !validInVisual;

			return;
		}

		exitedOnce = true;

		if (validate(event.target.value, name)) {
			markValidInVisual();

			return;
		}

		markInvalidInVisual();
	}

	function markValidInVisual() {
		validInVisual = true;
		tVisible = false;
	}

	function markInvalidInVisual() {
		validInVisual = false;
		tVisible = true;
	}
</script>

<Tooltip
	--tooltip-background-color={validInVisual === false ? "var(--error-color)" : ""}
	--tooltip-text-color={validInVisual === false ? "white" : ""}
	classes={wrapperClasses}
	text={tText}
	bind:visible={tVisible}
	bind:dontCloseOnHover={validInVisual}
	manual
>
	<div class="input-wrapper p-r">
		<input
			id={name}
			{name}
			{type}
			{inputmode}
			value={value ?? ""}
			disabled={disable}
			enterkeyhint={enterKeyHint ?? "next"}
			class="input-text b-box w-100 t-a-c {inputClasses ?? ''}"
			class:focus
			class:empty
			class:valid={validInVisual}
			class:invalid={validInVisual === false}
			class:disabled={disable}
			bind:this={input}
			on:focus={onFocus}
			on:beforeinput={onBeforeInput}
			on:input={onInput}
			on:blur={onBlur}
		/>

		<label class="input-label p-none p-a p-c f-w-800" for={name}>
			{title}{optional ? " - " + L("optional", $lang) : ""}
		</label>
	</div>
</Tooltip>
