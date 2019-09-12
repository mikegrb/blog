+++
title = "Search"
+++
<script async src="https://cse.google.com/cse.js?cx=009222613752762923653:lfpjzmqhnyn"></script>
<!-- Search box form -->
<form onsubmit="return executeQuery();" id="cse-search-box-form-id">
  <!-- This is the input searc box -->
  <input type="text" id="cse-search-input-box-id" size="25" autocomplete="off"/>
  <!-- This is the search button -->
  <input type="submit" value="Search"/>
</form>
<!-- End of search box form -->

<!-- REQUIRED: Attach the Google branding watermark to your search box. -->
<!-- WARNING: Branding attachment should be after transliteration -->
<!-- Set the "form" URL parameter to the id of the form containing the input
         search box.
     Set the "inputbox" URL parameter to the name or id of the query textbox.
         'q' will be used if it's not specified.
     Set the "lang" URL parameter to localize the branding for a specific
         language. Find the list of supported languages at
         http://developers.google.com/custom-search/docs/ref_languages.html
-->

<script src="//www.google.com/cse/brand?form=cse-search-box-form-id&inputbox=cse-search-input-box-id">
</script>
<!-- End of Google branding watermark -->

<!-- Element code snippet -->
<script>
  function executeQuery() {
    var input = document.getElementById('cse-search-input-box-id');
    var element = google.search.cse.element.getElement('searchresults-only0');
    if (input.value == '') {
      element.clearAllResults();
    } else {
      element.execute(input.value);
    }
    return false;
  }
</script>

<!-- Place this tag where you want the search results to render. -->
<div class="gcse-searchresults-only"></div>