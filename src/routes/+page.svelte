<script lang="ts">
  import ComboBox from "./ComboBox.svelte";
</script>

<ComboBox
  name="hello"
  on:before-options-update={async ({ detail }) => {
    try {
      if (detail.value === "") {
        detail.store.set([]);
      } else {
        // pretend the fetch result
        detail.store.set([{
            id: "1",
            name: 'foo'
        }, {
            id: "2",
            name: 'bar'
        }]);
      }
    } catch (error) {
      console.error("failed to update combobox options", error);
    }
  }}
  on:selected={({ detail }) => {
    //trying to reset the input and select value
    detail.store.set(undefined)
    detail.inputValue.set("")
    return;
  }}
></ComboBox>
