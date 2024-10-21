# magento
 
## Cara Install
1. Buat folder utk magento, lalu buka cmd dan jalankan
 ~~~ 
 composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition ./ 
 ~~~
2. buat database dg nama magento
3. download elasticsearch-7.17.0 dan jalankan
4. pergi ke \vendor\magento\framework\Image\Adapter\Gd2.php lalu ganti fungsi **validateURLScheme** jadi 
~~~ 
private function validateURLScheme(string $filename) : bool
  {
      $allowed_schemes = ['ftp', 'ftps', 'http', 'https'];
      $url = parse_url($filename);
      if ($url && isset($url['scheme']) && !in_array($url['scheme'], $allowed_schemes) && !file_exists($filename)) {
          return false;
      }

      return true;
  } 
~~~
5. Install Magento dg command
~~~ 
php bin/magento setup:install --base-url=http://localhost/magento/pub --db-host=localhost --db-name=magento --db-user=root --db-password= --admin-firstname=super --admin-lastname=admin --admin-email=admin@admin.com --admin-user=admin --admin-password=admin123 --language=en_US --currency=IDR --timezone=Asia/Jakarta --use-rewrites=1 --backend-frontname=admin --search-engine=elasticsearch7 --elasticsearch-host=localhost --elasticsearch-port=9200 
~~~
6. pergi ke \vendor\magento\framework\Interception\PluginListGenerator.php lalu ganti 
~~~
$cacheId = implode('|', $this->scopePriorityScheme) . "|" . $this->cacheId;
~~~
menjadi
~~~
$cacheId = implode('-', $this->scopePriorityScheme) . "-" . $this->cacheId;
~~~
7. jalankan command
~~~
php bin/magento setup:upgrade

php bin/magento setup:di:compile

php bin/magento setup:static-content:deploy -f

chmod -R 777 app/etc/* pub/static/ var/ generated/
~~~
8. lalu jalankan di cmd
~~~
php bin/magento config:set dev/static/sign 0

php bin/magento cache:flush

php bin/magento deploy:mode:set developer
~~~
9. buka \vendor\magento\framework\View\Element\Template\File\Validator.php lalu ganti fungsi **isPathInDirectories** menjadi
~~~
protected function isPathInDirectories($path, $directories)
    {
        $realPath = str_replace('\\', '/', $this->fileDriver->getRealPath($path));
        if (!is_array($directories)) {
            $directories = (array)$directories;
        }
        
        foreach ($directories as $directory) {
            if (0 === strpos($realPath, $directory)) {
                return true;
            }
        }
        return false;
    }
~~~
dan fungsi **isValid** menjadi
~~~
public function isValid($filename)
    {
        $filename = str_replace('\\', '/', $this->fileDriver->getRealPath($filename));
        if (!isset($this->_templatesValidationResults[$filename])) {
            $this->_templatesValidationResults[$filename] =
                ($this->isPathInDirectories($filename, $this->_compiledDir)
                    || $this->isPathInDirectories($filename, $this->moduleDirs)
                    || $this->isPathInDirectories($filename, $this->_themesDir)
                    || $this->_isAllowSymlinks)
                && $this->getRootDirectory()->isFile($this->getRootDirectory()->getRelativePath($filename));
        }
        return $this->_templatesValidationResults[$filename];
    }
~~~

10. pergi ke \vendor\magento\framework\App\StaticResource.php lalu ganti ``` DIRECTORY_SEPARATOR ``` menjadi ``` '/' ``` 
11. jalankan command
~~~
php bin/magento setup:static-content:deploy -f
~~~
12. jalankan command
~~~
php bin/magento module:disable Magento_AdminAdobeImsTwoFactorAuth

php bin/magento module:disable Magento_TwoFactorAuth
~~~
---
---
13. kalau bermasalah jalankan kembali
~~~
php bin/magento deploy:mode:set developer

php bin/magento setup:static-content:deploy -f
~~~


**TAMBAHAN PROMPT**
~~~
php bin/magento index:reindex
~~~