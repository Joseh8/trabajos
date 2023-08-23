<html>
<head>
<title>CARRO – Pasión por su casa</title>
<style type="text/css">
<!--
.Estilo1 {
	color: #FF0000;
	font-weight: bold;
}
-->
</style>
</head>
<body>
<div align="center">
  <h1><font color="#0000FF">CARRO</font> - Pasi&oacute;n por su casa </h1>
  <p>&nbsp;</p>
</div>
<?php
	function mes($n){
		$meses = array(
			1 => "Enero",
			2 => "Febrero",
			3 => "Marzo",
			4 => "Abril",
			5 => "Mayo",
			6 => "Junio",
			7 => "Julio",
			8 => "Agosto",
			9 => "Septiembre",
			10 => "Octubre",
			11 => "Noviembre",
			12 => "Diciembre");
		return $meses[$n];
	}
	function cadenafecha($fechaActual){
		//Separo las parte de la fecha y las almaceno en el array $fecha
		$fecha = split("/", $fechaActual);
		//Convierto el mes en un entero
		$mesActual = (intval($fecha[1]));
		/*Utilizo la función mes ya generada para obtener el mes y lo
		  paso a minúscula con strtolower*/
		$mesActual = strtolower(mes($mesActual));
		return "{$fecha[0]} de $mesActual de {$fecha[2]}";	
	}
function cli($clientes, $gastos, $clieimp, $cantcuo){
		static $a=0;// es la variable que nos permite contar los clientes
		/* Construimos el bucle que nos permitira recorre el array $gastos*/
		for($i=0;$i<sizeof($gastos);$i++){
			if ($gastos[$i] > 1000){ //Con esta condición solo trabajamos con los mayores a 1000
				$apedeu = $clientes[$i]["Apellido"];
				//$a++; // Incrementamos el valor de la variable que cuenta clientes
				echo ("\n<br><br>$i - {$clientes[$i]["Apellido"]} {$clientes[$i]["Nombre"]} <br> {$clientes[$i]["Localidad"]} ");			
				if (in_array($apedeu, $clieimp)!=false){
					echo "<br><span class=\"Estilo1\">Pero debe {$cantcuo[$i]} cuotas </span>";
				}
			}
		}
	}
	function formatCliente($indCliente){
		global $clientes;
		//Paso a mayúscula la primera letra con ucfirst porque hay una palabra
		$clientes[$indCliente]["Apellido"]=ucfirst($clientes[$indCliente]["Apellido"]);
		$clientes[$indCliente]["Nombre"]=ucfirst($clientes[$indCliente]["Nombre"]);
		//Utilizo ucwords porque puede haber más de una palabra.
		$clientes[$indCliente]["Localidad"]=ucwords($clientes[$indCliente]["Localidad"]);
	}
	  function cargarDatos($conexion){
		global $clientes, $gastos, $clieimp, $cantcuo;
		$sqltxt = "SELECT * FROM clientes";
		$resultado = mysql_query($sqltxt, $conexion);
		while($registro=mysql_fetch_array($resultado)){
			$clientes[$registro["indice"]]["Apellido"]=$registro["apellido"];
			$clientes[$registro["indice"]]["Nombre"]=$registro["nombre"];
			$clientes[$registro["indice"]]["Localidad"]=$registro["localidad"];
			$clientes[$registro["indice"]]["Telefono"]=$registro["telefono"];
		}
		$sqltxt = "SELECT * FROM gastos";
		$resultado = mysql_query($sqltxt, $conexion);
		while($registro=mysql_fetch_array($resultado)){
			$gastos[$registro["indice"]]=$registro["gasto"];
		}
		$sqltxt = "SELECT * FROM clieimp";
		$resultado = mysql_query($sqltxt, $conexion);
		while($registro=mysql_fetch_array($resultado)){
			$clieimp[$registro["indice"]]=$registro["nombre"];
		}
		$sqltxt = "SELECT * FROM cantcuo";
		$resultado = mysql_query($sqltxt, $conexion);
		while($registro=mysql_fetch_array($resultado)){
$cantcuo[$registro["indice"]]=$registro["cantidadCuotas"];
		}		
	}
	function grabarRegistro($conexion, $apellido, $nombre, $localidad, $telefono, $gasto, $ccuotas){
		$indice = indiceSiguiente($conexion);
		$sqltxt = "INSERT INTO clientes (indice, apellido, nombre, localidad, telefono) ";
		$sqltxt .= "VALUES ($indice, \"$apellido\", \"$nombre\", \"$localidad\", \"$telefono\")";
		mysql_query($sqltxt, $conexion);
		$sqltxt = "INSERT INTO gastos (indice, gasto) VALUES ($indice, $gasto)";
		mysql_query($sqltxt, $conexion);
		if($ccuotas>0){
			$sqltxt="INSERT INTO cantcuo (indice, cantidadCuotas) VALUES ($indice, $ccuotas)";	
			mysql_query($sqltxt, $conexion);
			$sqltxt="INSERT INTO clieimp(indice, nombre) VALUES ($indice, \"$apellido\")";	
			mysql_query($sqltxt, $conexion);
		}
	}
		function indiceSiguiente($conexion){
		$sqltxt = "SELECT MAX(indice) FROM clientes";
		$resultado = mysql_query($sqltxt, $conexion);
		$fila = mysql_fetch_row($resultado);
		return (++$fila[0]);
	}	
function formulario(){
		echo "<form name=\"form1\" method=\"post\" action=\"http://localhost/cursophp/tp12.php\">
  			<p>Apellido: 
			    <input name=\"apellido\" type=\"text\" id=\"Apellido\" size=\"30\" maxlength=\"30\">
			</p>
			  <p>Nombre: 
			    <input name=\"nombre\" type=\"text\" id=\"Nombre\" size=\"30\" maxlength=\"30\">
			</p>
			  <p>Localidad: 
			    <input name=\"localidad\" type=\"text\" id=\"Localidad\" size=\"30\" maxlength=\"30\">
			</p>
			  <p>Telefono: 
			    <input name=\"telefono\" type=\"text\" id=\"Telefono\" size=\"15\" maxlength=\"15\">
			</p>
			  <p>Gastos: 
			    <input name=\"cligastos\" type=\"text\" id=\"Cligastos\" size=\"10\" maxlength=\"10\">
			Cuotas impagas: 
			<input name=\"ccuotas\" type=\"text\" id=\"Ccuotas\" size=\"5\" maxlength=\"3\">
			</p>
			  <p>
			    <input name=\"Enviar\" type=\"submit\" id=\"Enviar\" value=\"Enviar\">
<input type=\"reset\" name=\"Submit2\" value=\"Restablecer\"> 			    
			</form><br><a href=\"tp12.php\">Home</a>";
	}
	if($mostrarFormulario=="si"){
		formulario();
	}else{
		//Conexion a la base de datos.
		$host="localhost";
		$usuario="root";
		$clave="";
		$conexion=mysql_connect($host, $usuario, $clave);
		mysql_selectdb("Carro", $conexion);
		if($REQUEST_METHOD=="POST"){
			//grabo y muestro.
 			grabarRegistro($conexion, $apellido, $nombre, $localidad, $telefono, $cligastos, $ccuotas);
			echo "Nuevo cliente grabado...<br>";
			echo "<b>Apellido:</b> $apellido<br>";
			echo "<b>Nombre:</b> $nombre<br>";
			echo "<b>Localidad:</b> $localidad<br>";
			echo "<b>Telefono:</b> $telefono<br>";
			echo "<b>Gastos:</b> $cligastos<br>";
			echo "<b>Cuotas adeudadas:</b> $ccuotas<br>";
			echo "<br><a href=\"tp12.php\">Home</a> | ";
			echo "<a href=\"tp12.php?mostrarFormulario=si\">Cargar nuevo cliente</a>";
			cargarDatos($conexion);
		}else{
			cargarDatos($conexion);
			//Paso a mayúcula la primera letra de cada palabra almacenada
			//en el arreglo mediante la función formatCliente.
			for($indice=0; $indice<count($clientes); $indice++)
				formatCliente($indice);
			/* Mostramos el texto Clientes del mes de */
			define("TEX1","Clientes del mes de ");
			echo TEX1;
			/* Mostramos MARZO */
			$n = 3;
			echo mes($n);
			/* LLAMAMOS A LA FUNCION QUE MUESTRA LOS NUMEROS Y LOS CLIENTES */
			$var="cli";
			$var($clientes, $gastos, $clieimp, $cantcuo);
			/*El gasto del mejor cliente */
define("TEX2","El mejor cliente es ");	
			asort($gastos);
			end($gastos);	
			echo "<br><br>". TEX2 . $clientes[key($gastos)]["Apellido"] . ", ";
			echo "{$clientes[key($gastos)]["Nombre"]} y gasto $" . number_format(current($gastos), 2, ",", ".");
			echo "<br><a href=\"tp12.php?mostrarFormulario=si\">Cargar nuevo cliente</a>";
			$fechaActual = "02/03/2007";	
			echo "<hr> <div align=\"center\">Resumen hecho el " . cadenafecha($fechaActual) . " </div>";	
		}
		mysql_close($conexion);
	}
?>
</body>
</html>
