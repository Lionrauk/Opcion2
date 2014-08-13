<?php

session_start();
$post_actual = $post;
include("consultas/general.php");
switch($_POST["action"])
{
   
    case "insert":
 
      
        if($_POST['CodigoSeguridad'] == $_SESSION['tmptxt']){
            
       
        // Create post object
        $new_entry = array();
        $new_entry['post_status'] = 'publish';
        $new_entry['post_title'] = $_POST['Nombre']." ".$_POST['Apellido'];
        $new_entry['post_type'] = 'contactenos';
       // $new_entry['post_category'] = array($_POST["IDProducto"]); // this is the line that doesn't work
        $new_entry['post_date'] = date("Y-m-d H:i:s");
        // Insert the post into the database
        if($id = wp_insert_post( $new_entry ))
        {
            
            $slug_pais = $_POST['Pais'];
            $slug_ciudad = $_POST['Ciudad'];
            
            wp_set_object_terms($id, $slug_pais, 'pais_ciudad', true);            // Add metadata to post
            wp_set_object_terms($id, $slug_ciudad, 'pais_ciudad', true);            // Add metadata to post
            
            update_post_meta($id, $shortname.'_nombre', $_POST['Nombre']);
            update_post_meta($id, $shortname.'_apellido', $_POST['Apellido']);
            update_post_meta($id, $shortname.'_direccion', $_POST['Direccion']);
            update_post_meta($id, $shortname.'_telefono', $_POST['Telefono']);
            update_post_meta($id, $shortname.'_email', $_POST['Email']);
            update_post_meta($id, $shortname.'_mensaje', $_POST['Observaciones']);
            
        $mensaje = "";
            foreach($_POST as $llave => $datos)
                if($llave != action)
                    $mensaje .= $llave.": ".$datos."\r\n";
            
            $headers = 'From: '.$_POST['Nombre'].' <'.$_POST['Email'].'>' . "\r\n";
          //  wp_mail( "fcaptuayo@simcolombia.com", ".:Mensaje CONTACTENOS HOTEL PORTON MEDELLIN:.", $mensaje, $headers );
           wp_mail( "reservasol@hotelportonbogota.com.co", ".:Mensaje CONTACTENOS HOTEL PORTON BOGOTA:.", $mensaje, $headers );
            
            echo '<script type="text/javascript">alert("Gracias, se ha registrado con \xe9xito");location.href="'.$_SERVER["HTTP_REFERER"].'"</script>';
            exit;

        }
         }
         else{
              echo '<script type="text/javascript">alert("Error en codigo de seguridad");location.href="'.$_SERVER["HTTP_REFERER"].'"</script>';
         }
      
    break;
    
}
$pais = get_terms( 'pais_ciudad',  array('orderby'=> 'count','hide_empty' => 0, 'parent' => 0 ));
/*BEGIN BANNER*/
$meta_query_banner[] = $where_banner= array(
        'key' => $shortname.'_tipo', 
        'value' => "publicidad_interna", 
        
    );
        $query_banner = array(
            'post_type' => array('banner'),
            'meta_query' => $meta_query_banner, 
            'posts_per_page' => 4
        );
$querypostbanner = new WP_Query($query_banner);
get_header();
?>
<link type="text/css" href="<?php echo get_bloginfo("template_url") ?>/js/calendar/jquery-ui-1.8.13.custom.css" rel="stylesheet" />
    <script type="text/javascript" src="<?php echo get_bloginfo("template_url") ?>/js/calendar/jquery-ui-1.8.13.custom.min.js"></script>
		<script type="text/javascript">
		jQuery(document).ready(function($){
      if ($(".date_picker" ).length != 0){
        $(".date_picker" ).datepicker({
          dateFormat: "yy-mm-dd",
          changeMonth: true,
          changeYear: true
          });
      }
		});
		</script>


<script type="text/javascript">
<?php
foreach($pais as $value_pais)
{
    $ciudades = NULL;
    $ciudades = get_terms( 'pais_ciudad',  array('orderby'=> 'count','hide_empty' => 0, 'parent' => $value_pais->term_id ));

?>
//defino una serie de varibles Array para cada país
var provincias_<?php echo $value_pais->slug?> = new Array(<?php foreach($ciudades as $value_ciudad){?>"<?php echo $value_ciudad->name?>",<?php }?>"Otro");
var provincias_<?php echo $value_pais->slug?>_value = new Array(<?php foreach($ciudades as $value_ciudad){?>"<?php echo $value_ciudad->slug?>",<?php }?>"Otro");

<?php
}
?>
    
//función que cambia las provincias del select de provincias en función del país que se haya escogido en el select de país.
function cambia_provincia(){
	//tomo el valor del select del pais elegido
	var pais
	pais = document.planes.Pais[document.planes.Pais.selectedIndex].value
	//miro a ver si el pais está definido
	if (pais != "") {
		//si estaba definido, entonces coloco las opciones de la provincia correspondiente.
		//selecciono el array de provincia adecuado
		mis_provincias=eval("provincias_" + pais)
                mis_provincias_value=eval("provincias_" + pais + "_value")
                //calculo el numero de provincias
		num_provincias = mis_provincias.length
		//marco el número de provincias en el select
		document.planes.Ciudad.length = num_provincias
		//para cada provincia del array, la introduzco en el select
		for(i=0;i<num_provincias;i++){
		   document.planes.Ciudad.options[i].value=mis_provincias_value[i]
		   document.planes.Ciudad.options[i].text=mis_provincias[i]
		}	
	}else{
		//si no había provincia seleccionada, elimino las provincias del select
		document.planes.Ciudad.length = 1
		//coloco un guión en la única opción que he dejado
		document.planes.Ciudad.options[0].value = "-"
	    document.planes.Ciudad.options[0].text = "-"
	}
	//marco como seleccionada la opción primera de provincia
	document.planes.Ciudad.options[0].selected = true
}
</script>


<script type='text/javascript' language="javascript">
    
    $(document).ready(function(){
              
                  $('#CodigoSeguridad').keyup(function() {
                 
                    var value = $('#CodigoSeguridad').val();
                   
                   var cadena = new String($('#CodigoSeguridad').val());
                   cadena = cadena.toLowerCase();
                   
                    
                    $.ajax( {
				"type" : "POST",
				"data" : { "Codigo" : cadena,"action":"Ingreso"},
				"url" : "http://www.hotelportonbogota.com.co/archivo/" ,
				"dataType" : "json" ,
				
				"beforeSend" : function(){
                                 //  $( "#RegistrarID" ).html( "<img src='img/ajax-loader.gif' />" );
                                 //  $( "#RegistrarID" ).css( "background","none" );
				},
				 
				"success" : function( data ){
					//data = data.column;
					
					 if( data.Existe == "S")
                              {
                                 if((value.length) >=8) {
                                 
                                  $("#CodigoSeguridad").css("border","1px solid red");
                                   }
                                   
                                   
                                 $("#CodigoBien").val(0);

                              }
                              else{
                                  $("#CodigoSeguridad").css("border","0px solid red");
                                  $("#CodigoBien").val(1);
                              }
				}
                            });	 
                    
                                            
                   });  
    
});


			function valida(form)
			{
                            
                            var enviar = 0;
                           
				       
                                  if( form.CodigoSeguridad.value == ''  )//nombre
				{
                                        enviar = 1;
                                        $("#CodigoSeguridad").css("border","1px solid red");
				}
                                else
                                    {
                                        if($("#CodigoBien").val() == 0)
                                        {
                                                     $("#CodigoSeguridad").css("border","1px solid red");
                                                     enviar = 1;
                                        }
                                        else
                                            {
                                                $("#CodigoSeguridad").css("border","0px solid red");
  
                                                
                                            }
                                    }
                                 if(enviar)
                                {
                                    
                                   //alert($("#CodigoBien").val());
                                   // $("html:not(:animated),body:not(:animated)").animate({ scrollTop: destination}, 0 );

                                    return false;
                                }
                                else
                                {
                                    
                                    return true;
                                }

				//return true;
			}
			
		</script>
</head>
	<body>
		<!--[if lt IE 8]>
			<p class=chromeframe>
				El navegador que esta usando es <em>obsoleto</em>, para tener la mejor experiencia en nuestro sitio web le recomendamos <a href="http://browsehappy.com/" target="_blank">cambiar su navegador</a> o instalar <a href="http://www.google.com/chromeframe/?redirect=true" target="_blank">Google Chrome Frame</a>
			</p>
		<![endif]-->
		<div class="centrar">
			<?php
                        include("componentes/redes.php");
                        ?>
			<div id="sombra">
				<?php
                                 include("componentes/header.php");
                                ?>
				
				<section id="contenedor">
					<article id="contenidos">
                                            <?php
                                            if (have_posts()):
                                                while (have_posts()) : the_post(); 
                                             ?>
                                            <?php if (has_post_thumbnail()): ?>
                                                            <?php the_post_thumbnail('planes-paquetes'); ?>
                                                        <?php endif; ?>
						<h1><?php $page_data = get_page(get_post_meta($post->ID, $shortname . '_modulo', true) );
                                               echo $page_data->post_title
                                                
                                                ?></h1>
						<div class="share">
							
							<!-- ACÁ VAN LOS PLUGINS PARA COMPARTIR EL CONTENIDO -->
								<!-- AddThis Button BEGIN -->
                                                                <div class="addthis_toolbox addthis_default_style ">
                                                                <a class="addthis_button_facebook_like" fb:like:layout="button_count"></a>
                                                                <a class="addthis_button_tweet"></a>
                                                                <a class="addthis_button_pinterest_pinit"></a>
                                                                 <a class="addthis_button_compact"></a>
                                                                <a class="addthis_counter addthis_pill_style"></a>
                                                                </div>
                                                                <script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=xa-51362b20396a43ec"></script>
                                                                <!-- AddThis Button END -->
								
						</div>
                                                <h2><?php the_title()?></h2>
                                                <p>
                                                    
                                                    <?php the_content()?>
                                                </p>
                                            <?php
                                               endwhile;
                                             endif;
                                            ?>
                                                
                                                 <?php
                                                 
                                                 
 
                                                 $query_Mensajes = array(
                                                                    'post_type' => 'mensajes',
                                                                  'posts_per_page' => -1,
                                                                );
                                                            

                                                        $wp_query_planes = new WP_Query($query_Mensajes);
                                                       // wp_reset_query();
                                                        
                                                        if ($wp_query_planes->have_posts())
                                                        {
                                                            while ($wp_query_planes->have_posts())
                                                            {
                                                               
                                                                $wp_query_planes->the_post();
                                                                
                                                              
                                                                
                                                                
                                                              
                                         //   $arraymensajes[$post->ID]=get_post_meta($post->ID, $shortname . '_identificador', true);
//                                                                $arraymensajes[$post->ID]= $post->post_title;
                                                                 $arraymensajes[get_post_meta($post->ID, $shortname . '_identificador', true)]= $post->post_title;
                                                                ?>
                                                               
                                                                <?php
                                                            }
                                                        }
                                                     
                                                        ?>
                                              <?php
                                            //  print_r($arraymensajes);
                                              
                                             
                                              
                                              ?>
						<div class="contactenos">
							<form action="" id="planes" name="planes" class="validarform" method="POST" onSubmit="return valida(this)" >
								<!-- <h4>Contáctenos</h4> -->
								<h5><?php  echo $arraymensajes[mensaje_de_envio_comentario];?></h5>
								<div>
									<label ><?php  echo $arraymensajes[nombre];?></label>
									<input type="text" name="Nombre" class="mandatory" />
									<label ><?php  echo $arraymensajes[apellido];?></label>
									<input type="text" name="Apellido" class="mandatory" />
								</div>
								<div>
									<label ><?php  echo $arraymensajes[direccion];?></label>
									<input type="text" name="Direccion" class="mandatory" />
									<label ><?php  echo $arraymensajes[telefono];?></label>
									<input type="text" name="Telefono" class="mandatory" />
								</div>
								<div>
									<label for=""><?php  echo $arraymensajes[pais];?></label>
									<select name="Pais" id="Pais" class="mandatory" onChange="cambia_provincia()">
										<option value="">Seleccione</option>
                                                                                <?php
                                                                                foreach($pais as $value_pais)
                                                                                {
                                                                                ?>
                                                                                <option value="<?php echo $value_pais->slug?>"><?php echo $value_pais->name?></option>
                                                                                <?php
                                                                                }
                                                                                ?>
									</select>
									<label for=""><?php  echo $arraymensajes[Ciudad];?></label>
									<select name="Ciudad" id="Ciudad" class="mandatory">
										<option value="">Seleccione</option>
									</select>
								</div>
								<div>
									<label >E-mail</label>
									<input type="text" name="Email" class="mandatory valmail" />
								</div>
								<div>
									<label ><?php  echo $arraymensajes[observaciones];?></label>
									<textarea name="Observaciones" id="Observaciones" cols="30" rows="10" class="mandatory"></textarea>
								</div>
                                                                
                                                               
                                                                 <div>
                            <label >Captcha:</label>
                            
                            <input class="cap4 obligatordio" id="CodigoSeguridad" name="CodigoSeguridad" />
                          
                              </div>
                                              <div>                    
                            <img style="  margin: -15px 0 0 93px;" src="<?php echo get_bloginfo('template_url') ?>/captcha/captcha.php" width="200" height="40" vspace="3" />
                            
                                                                </div>
								<div>
									<button rel="planes" class="EnviaForm" ><?php  echo $arraymensajes[enviar];?></button>
								</div>
                                                                <input type="hidden" name="action" value="insert" />
                                                                <input type="hidden" id="CodigoBien" name="CodigoBien" value="0" />
							</form>
						</div>
						
						
					</article>
					<aside id="sidebar">
						 <?php
                                                include("componentes/form_reservas.php");
                                                ?> 
						<ul>
							<!-- <lh>Planes Relacionados</lh> -->
                                                        <?php
                                                        $where1= array(
                                                                'key' => $shortname.'_modulo', 
                                                                'value' => $post_actual->ID, 

                                                            );
                                                        $meta_query[] = $where1; 
                                                             if (isset($meta_query)) {
                                                                $query = array(
                                                                   'orderby' => 'menu_order', 
                                                                    'order' => 'ASC',
                                                                    'post_type' => array('planes'),
                                                                    'meta_query' => $meta_query, 
                                                                    'posts_per_page' => 10,
                                                                );
                                                            }

                                                        $wp_query = new WP_Query($query);
                                                        wp_reset_query();
                                                        
                                                        if ($wp_query->have_posts())
                                                        {
                                                            while ($wp_query->have_posts())
                                                            {
                                                                $wp_query->the_post();

                                                                ?>
                                                                <li><a href="<?php the_permalink()?>" title="<?php the_title()?>"><?php the_title()?></a></li> 
                                                                <?php
                                                            }
                                                        }
                                                        ?>
							
						</ul>
						<?php
						include("componentes/banner_publicidad.php");
                                                ?>
					</aside>
				</section>
				
				<?php
                                include("componentes/nav_footer.php")
                                ?>
			</div>
			
		</div>
<?php
include("componentes/footer.php");
?>
