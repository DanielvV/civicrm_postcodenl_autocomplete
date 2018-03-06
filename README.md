CiviCRM PostcodeNL autocomplete
===============================

Call `https://example.com/civicrm_postcodenl_autocomplete?postcode=&huisnummer=` after installing this module in Drupal.

Dependencies:
- [civicrm](https://civicrm.org/)
- [webform](https://www.drupal.org/project/webform)
- [webform_civicrm](https://www.drupal.org/project/webform_civicrm)
- [org.civicoop.postcodenl](https://github.com/CiviCooP/org.civicoop.postcodenl) (CiviCRM extension)



*Example usage in Drupal Webform:*

```
<script>
(function($) {

$postal_code_field = $('.webform-client-form #edit-submitted-civicrm-1-contact-1-address-postal-code');
$housenumber_field = $('.webform-client-form #edit-submitted-civicrm-1-contact-1-address-street-number');
$housenumber_suffix_field = $('.webform-client-form #edit-submitted-civicrm-1-contact-1-address-street-unit');
$street_field = $('.webform-client-form #edit-submitted-civicrm-1-contact-1-address-street-name');
$city_field = $('.webform-client-form #edit-submitted-civicrm-1-contact-1-address-city');

$postal_code = '';
$housenumber = '';
$postal_code_prev = '';
$housenumber_prev = '';
$street = '';

$street_field.attr("disabled", "disabled");
$city_field.attr("disabled", "disabled");
$housenumber_suffix_placeholder = $housenumber_suffix_field.attr("placeholder");

$busy = [];
$empty = true;
$clear_postal_code = false;
$clear_housenumber = false;
$dontrun = false;

getAndSetAddress = function ($postal_code, $housenumber, $field) {
    $.get(
        "https://example.com/civicrm_postcodenl_autocomplete",
        {
            postcode: $postal_code,
            huisnummer: $housenumber
        },
        function(data) {
            setAddress($postal_code, $housenumber, $field, data);
        },
        'json'
    );
}

setAddress = function ($postal_code, $housenumber, $field, $result) {
    if ($result) {
        $postal_code = $result.postcode_nr + ' ' + $result.postcode_letter;
        $postal_code_prev = $postal_code;
        $postal_code_field.val($postal_code);
        $street = $result.adres;
        $street_field.val($street);
        $city_field.val($result.woonplaats);
        $empty = false;
        $clear_postal_code = false;
        $clear_housenumber = false;
    } else {
        unsetAddress($postal_code, $housenumber);
    }
}

$(function() {
  // Handler for .ready() called.
  jQuery(".webform-client-form input.button-primary").click(function(e){
      if ($empty || $busy.length !== 0) {
          $dontrun = true;
      } else {
          $dontrun = false;
          $street_field.removeAttr("disabled");
          $city_field.removeAttr("disabled");
          $housenumber_suffix_field.val($housenumber_suffix_field.val().toUpperCase());
      }
  });
});

$(".webform-client-form").submit(function(e){
    if ($dontrun) {
        e.preventDefault();
    }
});

unsetAddress = function ($postal_code, $housenumber) {
    if (!$empty) {
        $empty = true;
        $street_field.val('');
        $city_field.val('');
    }
    if ($clear_postal_code) {
        $postal_code_field.val('');
        $clear_postal_code = false;
    } else if ($clear_housenumber) {
        $housenumber_field.val('');
        $clear_housenumber = false;
    }
}

runLater = function ($field) {
    $busy.push(true);
    setTimeout(function(){
        $busy.pop();
        if ($busy.length === 0) {
            if ($field == 'housenumber_suffix') {
                if ($housenumber_suffix_field.val() == '') {
                    $housenumber_suffix_field.css("text-transform", "none");
                    $housenumber_suffix_field.attr("placeholder", $housenumber_suffix_placeholder);
                } else {
                    $housenumber_suffix_field.css("text-transform", "uppercase");
                    $housenumber_suffix_field.attr("placeholder", '');
                    $housenumber_suffix_field.val($housenumber_suffix_field.val().toUpperCase());
                }
            }
            checkAddress($field);
        }
    }, 500);
}

runNow = function ($field) {
    if ($field == "postal_code" && $housenumber_field.val() != '') {
        $clear_postal_code = true;
        $clear_housenumber = false;
    } else if ($field == "housenumber" && $postal_code_field.val() != '') {
        $clear_housenumber = true;
        $clear_postal_code = false;
    }
}

checkAddress = function ($field) {
    if (typeof $postal_code == 'undefined') {
        $postal_code_prev = false;
    } else {
    }
    if (typeof $housenumber == 'undefined') {
        $housenumber_prev = false;
    } else {
    }
    if ($postal_code_prev && $housenumber_prev) {
        $prev = true;
    } else {
        $prev = false;
    }

    $postal_code_prev = $postal_code;
    $housenumber_prev = $housenumber;
    $postal_code = $postal_code_field.val();
    $housenumber = $housenumber_field.val();

    if ($postal_code != $postal_code_prev || $housenumber != $housenumber_prev) {
        if ($housenumber && $postal_code && $postal_code.length >= 6) {
            getAndSetAddress($postal_code, $housenumber, $field);
        } else {
            unsetAddress($postal_code, $housenumber);
        }
    } else if ($empty) {
        unsetAddress($postal_code, $housenumber);
    }
}

$postal_code_field.keyup(function() {
    runLater('postal_code');
});
$postal_code_field.blur(function() {
    runNow('postal_code');
    runLater('postal_code');
});
$postal_code_field.focus(function() {
    $clear_postal_code = false;
    $clear_housenumber = false;
});
$housenumber_field.keyup(function() {
    runLater('housenumber');
});
$housenumber_field.blur(function() {
    runNow('housenumber');
    runLater('housenumber');
});
$housenumber_field.focus(function() {
    $clear_postal_code = false;
    $clear_housenumber = false;
});
$housenumber_suffix_field.keyup(function() {
    runLater('housenumber_suffix');
});

runNow('postal_code');
runLater('postal_code');

})(jQuery);
</script>
```
