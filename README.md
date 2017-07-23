# jQuery File Upload Plugin - PHP Snapshot

This is a standalone snapshot of [jQuery-File-Upload](//github.com/blueimp/jQuery-File-Upload) v9.18.0 tailored to PHP web servers (Apache/Nginx).

This version does not request assets from other domains and is supposed to work nearly "out-of-the-box" on PHP enabled web servers. The permissions still need fixing because Linux.

Please visit the original site for:

- node.js, and other web server technologies.
- [Features list.](//github.com/blueimp/jQuery-File-Upload)
- [Wiki](//github.com/blueimp/jQuery-File-Upload/wiki).
    - [Frequently-Asked-Questions](//github.com/blueimp/jQuery-File-Upload/wiki/Frequently-Asked-Questions).
    - Non-Apache [setup guide](//github.com/blueimp/jQuery-File-Upload/wiki/Setup).
    - [Security Considerations](//github.com/blueimp/jQuery-File-Upload/wiki/Security).
- Original [Demo](//blueimp.github.io/jQuery-File-Upload/).
- Requirements. (Fully included in this version.)
- Licensing information (MIT).

### Supported features


Failed to resize image (original, thumbnail)

## Contributing and Issues

- Please submit all **bug fixes** and **new features** to the main repo using [pull requests](//github.com/blueimp/jQuery-File-Upload/pulls).

## Support

- The main repo for this project is actively maintained, but there is no official support channel.  
- If you have a question that another developer might help you with, please post to [Stack Overflow](http://stackoverflow.com/questions/tagged/blueimp+jquery+file-upload) and tag your question with `blueimp jquery file upload`.
- There is no support for this standalone snapshot besides the Installation Guide below.
- The error `Failed to resize image (original, thumbnail)` gets printed to the screen when uploading images (will not fix).

## Installation Guide

Install [Ubuntu 16.04 LTS](https://www.ubuntu.com/download/server) or another operating system. The instructions below are for Ubuntu/Debian.

```
#Login
Username: `user`
Password: `Password1`

#switch to root
sudo su -

#update and install dependencies
apt-get update
apt-get upgrade -y
apt-get install apache aria2 zip nmap -y

#configure Apache

# TODO: this section

#Check to make sure Apache is working on port 80.
nmap localhost

#install 
cd /var/www/html

#cleanup the root of the web server
ls -la
rm -r *
ls -la

#download software
aria2c https://github.com/gdiaz384/jQuery-File-Upload/releases/download/v9.18.0-standalone/jQuery-File-Upload-9.18.0-standalonePHP.zip

#extract software
unzip jQuery-File-Upload-9.18.0-standalonePHP.zip

#move the software to home directory as a backup
mv jQuery-File-Upload-9.18.0-standalonePHP.zip ~

#fix permissions
chmod  0755 -R ../html
chown www-data:www-data -R ../html

```

Should work now. Check using another computer at http://192.168.0.253 and try uploading a file
The path from the web server's root where the files are stored is `/server/php/files`.
Assuming a web server root of `/var/www/html`, the complete path is `/var/www/html/server/php/files` and attempting to change that breaks everything, so don't. Use directory junctions, `ln --help`, instead or mount the `files` directory using NFS or fdisk.

## On Disabling the "Delete" Functionality

To remove the ability of users to delete uploaded files:

`nano /var/www/html/server/php/UploadHandler.php`
`CTRL + w` 
`public function delete`

And change the following function from:

```
    public function delete($print_response = true) {
        $file_names = $this->get_file_names_params();
        if (empty($file_names)) {
            $file_names = array($this->get_file_name_param());
        }
        $response = array();
        foreach ($file_names as $file_name) {
            $file_path = $this->get_upload_path($file_name);
            $success = is_file($file_path) && $file_name[0] !== '.' && unlink($file_path);
            if ($success) {
                foreach ($this->options['image_versions'] as $version => $options) {
                    if (!empty($version)) {
                        $file = $this->get_upload_path($file_name, $version);
                        if (is_file($file)) {
                            unlink($file);
                        }
                    }
                }
            }
            $response[$file_name] = $success;
        }
        return $this->generate_response($response, $print_response);
    }
```

To:

```
 public function delete($print_response = true) {
        return 'Delete action not supported.';
    }
```
`CTRL + o`
`CTRL + x`

Alternatively, use PHP comments: `/*  This is a comment  in PHP */`.

This will not remove the client-side "delete" buttons. They, and client side form submitions, will just not work anymore. 

To remove the buttons:

`nano /var/www/html/index.html`

Delete the following section:
```
    <button type="button" class="btn btn-danger delete">
        <i class="glyphicon glyphicon-trash"></i>
        <span>Delete</span>
    </button>
```
`CTRL + o`
`CTRL + x`

`CTRL + k` deletes an entire line at a time, instead of waiting for backspace. 
Alternatively, comment it out using the following syntax: 
```
<!--  This is an
html comment.  -->
```

Might want to also remove the checkboxes which, while fun to play with, are now pointless.
From:  `<input type="checkbox" class="toggle">`
To: `<!--<input type="checkbox" class="toggle">-->`

Under the `<!-- Display files available for download -->` heading, change:

```
<!-- Display files available for download -->

[...]

            {% if (file.deleteUrl) { %}
                <button class="btn btn-danger delete" data-type="{%=file.deleteType%}" data-url="{%=file.deleteUrl%}"{% if (file.deleteWithCredentials) { %} data-xhr-fields='{"withCredentials":true}'{% } %}>
                    <i class="glyphicon glyphicon-trash"></i>
                    <span>Delete</span>
                </button>
                <input type="checkbox" name="delete" value="1" class="toggle">
            {% } else { %}
                <button class="btn btn-warning cancel">
                    <i class="glyphicon glyphicon-ban-circle"></i>
                    <span>Cancel</span>
                </button>
            {% } %}
        </td>
    </tr>
{% } %}
</script>
```

To:
```
<!-- Display files available for download -->

[...]

            {% if (file.deleteUrl) { %}
<!---           <button class="btn btn-danger delete" data-type="{%=file.deleteType%}" data-url="{%=file.deleteUrl%}"{% if (file.deleteWithCredentials) { %} data-xhr-fields='{"withCredentials":true}'{% } %}>
                    <i class="glyphicon glyphicon-trash"></i>
                    <span>Delete</span>
                </button>
                <input type="checkbox" name="delete" value="1" class="toggle"> --->
            {% } else { %}
                <button class="btn btn-warning cancel">
                    <i class="glyphicon glyphicon-ban-circle"></i>
                    <span>Cancel</span>
                </button>
            {% } %}
        </td>
    </tr>
{% } %}
</script>
```

If you would like to keep the pointless checkboxes, put the final `--->` one line above after `</button>` to end the HTML comment sooner:
Example: `</button> -->`

