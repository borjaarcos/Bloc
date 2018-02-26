# Bloc
/*index.php*/
<?php

    require 'sys/DB.php';
    define('CONF',__DIR__.'/Config.json');
      
    use App\Sys\DB;
    //connection gdb
    
    $name= "Paco";
    $gdb=DB::getInstance();
    
    $query= "INSERT INTO user VALUES (5, 1, :username, :rol, :password)";
    
    
    $sentencia = $gdb->query($query);
    
    $gdb->bind(':username', 'name');
    $gdb->bind(':rol', 'rol');
    $gdb->bind(':password', 'pass');
    
    $gdb->execute();
    $gdb->resultSet();
    
    $gdb->rowCount();
    $gdb->resultSet();
    $gdb->single();
    
    
    /*DB.php*/
    <?php

/* 
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */

namespace App\Sys;

require 'Helper.php';

use App\Sys\Helper;

class DB extends \PDO{
    private $stmt;
    static private $_instance=null;
    
    static function getInstance(){
        if(!(self::$_instance instanceof self)){
            self::$_instance=new self();
        }
        return self::$_instance;
    }

    function __construct(){
       
        $dbconf= Helper::getConfig();
        
        $dsn=$dbconf['driver'].':host='.$dbconf['dbhost'].';dbname='.$dbconf['dbname'];
        $usr=$dbconf['dbuser'];
        $pwd=$dbconf['dbpass'];
        
        parent::__construct($dsn,$usr,$pwd);
    }

    
    public function query($query){
       $this->stmt= $this->prepare($query);
       var_dump($query);
    }
    
    public function bind($parametro, $var) {
        switch (true){
            case is_int($var): $type = \PDO::PARAM_INT;
                break;
            case is_bool($var): $type = \PDO::PARAM_BOOL;
                break;
            case is_null($var): $type = \PDO::PARAM_NULL;
                break;
            default: $type = \PDO::PARAM_STR;
        }
        $this->stmt->bindValue($parametro, $var, $type);
    }
     public function execute() {
        $result = $this->stmt->execute();
      
        var_dump($result);
    }
    public function resultSet(){
        $resultado = $this->stmt->fetchAll(self::FETCH_ASSOC);
       
        print_r($resultado);
    }
    
    public function single(){
        $resultado = $this->stmt->fetch(self::FETCH_ASSOC);
       
        print_r($resultado);
    }
    public function rowCount(){
        $cuenta = $this->stmt->rowCount();
        
        var_dump( $cuenta);
        
    }
    /*
    public function lastInsertId(){
        $last = DB::$_instance->lastInsertId();  

        return $last;
    }
      */
     
    
    public function beginTransaction()
    {
        if (!$this->stmt->transactionCounter++) {
            return parent::beginTransaction();
        }
        $this->stmt->exec('SAVEPOINT trans'.$this->stmt->transactionCounter);
        return $this->stmt->transactionCounter >= 0;
    }
    
    function endTransaction()
    {
        if (!--$this->stmt->transactionCounter) {
            return parent::commit();
        }
        return $this->stmt->transactionCounter >= 0;
    } 
    
    function cancelTransaction()
    {
        if($this->stmt->transactionCounter >= 0)
        {
            $this->stmt->transactionCounter = 0;
            return parent::rollback();
        }
        $this->stmt->transactionCounter = 0;
        return false;
    } 
    
    function debugDumpParams()
    {
        return $this->stmt->debugDumpParams();
    }
}

