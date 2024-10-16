**Stuff to improve/evaluate in proc**

*Priority*

1.  Bug! **Disalowed Decryptor**. Do not allow the current user to try to decrypt a content the user is not a recipient of. Currently, this is even causing a js error (Uncaught (in promise) Error: readPrivateKey: must pass options object containing `armoredKey` or `binaryKey`.) and prevents the user to correctly edit a form with content for which the user does not have accees to. The problem only happens in edit mode, in view mode it behaves well, displaying only the labels of the content.

2.  Bug! **Innefective API tool**. Fix changeProcElementSettings. It is capable of changing the elemet settings array, but the changed value will not take effect once the element is already rendered at this moment. (Retest! It is possible that this is already fixed at version 10.1.81).

3.  Bug! **Failed Text/File Decryption**. When a text field is decrypted and re-encrypted, the decryption of the NEXT or PREVIOUS text/file field does not work. The password dialog opens empty (despite of cache being enabled or not), and yet the typed in password will not work (by clicking Open nothing happnes, and there is no error in the cosole). It will only work when the fields are first, all of them, decrypted, and then encrypted one by one or if the fields were already decrypted at current page load. (Workaround: set the text field to automatically decrypt the next text field by checking "Trigger decryption of another encrypted field").

4.  **Be Smart**. Do not show Decrypt if the current user is not a recipient.

5.  Bug! **Workflow Failure**. When a user is not the author of a proc file, the simple update button should instead create a new proc entity. Currently it is just going to fail.

6.  Bug! **Fresh Install**. When fresh installing, proc will try to use a <private://> stream wrapper. And this does not exist in a Drupal fresh installation. When trying to encrypt, and error will happen: "php Error Drupal\Core\File\Exception\InvalidStreamWrapperException: Invalid stream wrapper: private://proc/lkrZQf-eHc9Tnmn4ZeyRrQB-_V1IxexQu-596oXSTMg.json in Drupal\file\FileRepository→writeData()"

7. Bug! Repeated Proc ID Cache Invalidation Failure. Steps to reproduce: 1. Install proc (having no proc table before), 2. Create keys, they will be given key id number #1. 3. Encrypt a content (it will be given proc ID number #2), 4. Succesfully decrypt the content. 5. Now, uninstall proc, removing the proc table and without clearing the browser cache and using the same browser, repeat 1-3, 6. Try to decrypt the proc with ID number #2 and get the error “Unable to decrypt the content. Make sure you have entered the right passphrase.” in the browser’s console. Workaround: clear browser cache.

8. Bug! At the "admin/structure/types/manage/<type>/fields/add-field" every Entitty Reference Field type (Content, User, Profile, Taxonomy Term) appears with the description of the Proc Type: "An entity field containing a proc enabled entity reference".

9. Bug! TO: field is only working when there is also a CC field defined. **fixed**

10. Bug! TO: and CC: fields fails to collect multiple user IDs. Only the first user ID is being taken.

11. Create unit tests.

*Security coverage requirement*

1.  Add examples of usage (private message, internal communication, static list of recipients, perhaps as exported configurations as submodules).

2.  Consider applying this: "Writing a conceptual overview of the purpose of the code and its general methodology in the @file section is helpful to future developers. It may even help designing the code during the planning stages."

3.  Mention in README.md hooks and plugins.

4.  Check how/if proc permissions are being used. Use field_permission_example as reference. **Wip**

5.  Verify access to view the label. Operation: 'view label'

6.  <https://www.drupal.org/docs/drupal-apis/routing-system/structure-of-routes> → **_csrf_token**: should be used and set to 'TRUE' for any URLs that perform actions or operations that do not use a form callback. See [Access checking on routes](https://drupal.org/node/3048359) for details.

7.  Create access check services as per: "Additionally, you have to add the access class as a service by adding an entry in example.services.yml." (<https://www.drupal.org/docs/8/api/routing-system/access-checking-on-routes/advanced-route-access-checking>) **Wip**

8.  Consider validating the proc ID slugs in routes by using regular expressions, as it is shown here: <https://www.drupal.org/node/2399239> (`name``:``'[a-zA-Z]+'`)

9.  Restrict anonymous user access to [/api/proc/getpubkey/1,2,3/user_id](http://127.0.0.1:8181/proclab/proclab/web/api/proc/getpubkey/1,2,3/user_id) by also requiring 'View proc entity' permission. Implement it with another Access class and service.

10. Review effectiveness of switching published/unpublished proc content. Currently this difference is not being used for any explicit visibility control.

11. Implement *hook_update_last_removed* in .install for removing the empty hook update.

12. Check access to admin/content/procs.

13. Review the "_entity_access: 'proc.view'" for '/proc/{proc}/details'.

14. Review the usage of ProcAccessControlHandler::checkAccess(). View, update and delete parameters must be passed with the route entry.

15. Implement .view, .update and .delete operations for checking access. Currently they exist but they are not being used.

16. Phaseout _proc_get_keys(). Replace it by a trait.

17. Adjust "created" timestamp to "changed" where is is mentioned as such. Example: `proc_recipients_pubkeys_changed' => json_encode($context['created'])`

*General improvements*

1.  Improve setMetadataMessage. It has too many arguments. Some can be put toghether as values of an array.

2.  Extract and add more generic methods to ProcOpFormBase (it contains only one base method, currently).

3.  Unify "'library' => [" (used 5 times, currently) in a single base method to be called from everywhere it is used.

4.  Check if settings summary of the widget covers all settings available in the widget.

5.  Check if global configuration "Enable local password cache for fields" is being used. If it is not, remove it.

6.  Test adequacy for the 3 cardinality types (unlimited, limited to 1, limited to 1+) and the 2 field types (files, text) and other less used combinations of configurations.

7.  Rewrite this message using megabytes instead of bytes: "Maximum file size allowed: 2000000 bytes".

8.  If a proc field does not have recipients defined, create an error message for the user and a log it. Currently it will just return a "403 (Forbidden)" in the console because of trying to access "proc/add/?...". The error caused therefore by trying to encrypt for no one.

9.  Create js module for not repeating "const i in navigator" (it happens currently twice)

10. Move "= $module_path . '/templates/proc" to a helper method removing repeated code.

11. "Encrypt" checkbox label in the widget for text fields should also overwrite the global configuration for the encrypt button label. Currently, this works only for the "Encrypt" button in proc **file** fields.

12. Define better permission for the path "/proc/{proc}/details". Currently it is controlled as "_entity_access: 'proc.view'".

13. Move _proc_get_csv_argument functionality out of functions file. Consider putting it in the FormBase class.

14. Consider adding a view with json output listing users with keuys and only users wuth keys for fast deployment as fetcher URL in proc fields.

15. Add a descryption to the "Type of identifier" field in the widget.

16. Merge TO and CC fields in the field widget into a single multivalued field approach, for allowing building up the set of recipients from any number of user reference fields. This could work with a CSV format and validated against the list of existing user reference fields.

*Re-encryption (encryption update)*

1.  Create a new multivalued field for wished set of recipients. **done**

2.  Allow this field (Wished recipient name) to be changed at the edit form of the proc entity **done**

3.  Store as metadata of a cipher proc the full URL that originated the set of recipoients. **done**

4.  Replace ProcController::extractProcIdsFromPath by ProcCsvTrait::getCsvArgument(). **done**

5.  Phaseout _get_recipient_ids(). **done**

6.  Phaseout _proc_common_get_ciphers() **done**

7.  Phaseout isUserOwnerOrAdmin() **done**

8.  Allow the decryption of multiple contents in standalone mode. This will be reused for the next item (5). **done**

9.  Phaseout _proc_common_get_decryption_form_data(). **done**

10.  Create a new route proc/\<proc-IDs-CSV\>/update and its ad hoc JS library (based on d7 **proc.update_protected.js**) for performing the re-encryption of the PROCs listed at \<proc-IDs-CSV\> according to the wished lists of recipients in each Proc. **done**
 
*For giving back to the Community*

1\. Report typo error in example module: "In drupal, a route is a path ***which is returns** *some response." should be "In drupal, a route is a path ***which returns*** some response.".
