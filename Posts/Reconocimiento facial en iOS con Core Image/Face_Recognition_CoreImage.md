En 2010, Apple compró una empresa especializada en algoritmos de reconocimiento facial (Polar Rose - Suecia). Gracias a esta adquisición y a partir de **iOS 5**, Apple hizo pública una API incluida en Core Image donde podremos hacer cosas como esta:

http://www.youtube.com/watch?feature=player_embedded&v=0QBLKBYrgvk

Así que vamos a entrar en materia y veremos como poder implantarlo en nuestro propio proyecto.

## Empezamos

*   Vamos a crear un nuevo proyecto en Xcode, para este ejemplo he utilizado "*Single View Application*" (DetectorCaras) 
*   Importamos los frameworks *Core Image* y *QuartzCore* al fichero de implementación de *ViewController* (ViewController.m) 

[<img src="http://objective-c.es/wp-content/uploads/2012/11/Link_Binary_Libraries.jpeg" alt="Link binary with libraries" title="Link binary with libraries" width="261" height="148" class="aligncenter size-full wp-image-293" />][1]

    #import <CoreImage/CoreImage.h>
    #import <QuartzCore/QuartzCore.h>
    

* Añadimos al proyecto una imagen en la cual deberá aparecer mínimo una cara. 
* Incluimos el método *faceDetector* (lo podéis llamar como queráis) para añadir la imagen,seleccionada anteriormente, a la pantalla. 

<!-- -->


    -(void)faceDetector {
        // Carga la imagen de la cara
        UIImageView *imagen = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"megan.jpg"]];
    
        // La añadimos a nuestra vista
        [self.view addSubview:imagen];
        
        // Llamamos al método markFaces para poder detectar los rostros y sí los detecta pintarlos en pantalla
        [self markFaces:imagen];
    }
    

*   Ahora vamos a añadir el método markFaces: con el que detectaremos si existen caras en la imagen.

<!-- -->

    -(void)markFaces:(UIImageView *)imagenCara {
    
        CIImage *imagen = [CIImage imageWithCGImage:imagenCara.image.CGImage];
        // Para el detector, utilizamos la constante "CIDetectorAccuracyHigh" la cual nos proporciona una precisión mejor pero requiere más tiempo de procesado
        // Más info en http://developer.apple.com/library/ios/#documentation/CoreImage/Reference/CIDetector_Ref/Reference/Reference.html
        CIDetector *detector = [CIDetector detectorOfType:CIDetectorTypeFace context:nil options:[NSDictionary dictionaryWithObject:CIDetectorAccuracyHigh forKey:CIDetectorAccuracy]];
        
        // Creamos un array con todas las caras detectadas en la imagen
        NSArray *features = [detector featuresInImage:imagen];
        
        // Hacemos un bucle por si detecta más de un rostro en la imagen
        for (CIFaceFeature *faceFeature in features) {
            // Obtener el ancho de la cara
            CGFloat faceWidth = faceFeature.bounds.size.width;
        
            // UIView usando las dimensiones de la cara detectada
            UIView *faceView = [[UIView alloc] initWithFrame:faceFeature.bounds];
        
            // Añadimos un borde alrededor del UIView
            faceView.layer.borderWidth = 1;
            faceView.layer.borderColor = [[UIColor blueColor] CGColor];
        
            [self.view addSubview:faceView];
        
            // Ojo derecho
            if (faceFeature.hasRightEyePosition) {
                // Creamos una UIView con el tamaño del ojo derecho
                UIView *rightEye = [[UIView alloc] initWithFrame:CGRectMake(faceFeature.rightEyePosition.x - faceWidth * 0.15, faceFeature.rightEyePosition.y - faceWidth * 0.15, faceWidth * 0.3, faceWidth * 0.3)];
        
                rightEye.backgroundColor = [[UIColor redColor] colorWithAlphaComponent:0.4];
                rightEye.center = faceFeature.rightEyePosition;
        
                rightEye.layer.cornerRadius = faceWidth * 0.15;
                [self.view addSubview:rightEye];
            }
        
            // Ojo izquierdo
            if (faceFeature.hasLeftEyePosition) {
                // Creamos una UIView con el tamaño del ojo izquierdo
                UIView *leftEye = [[UIView alloc] initWithFrame:CGRectMake(faceFeature.leftEyePosition.x - faceWidth * 0.15, faceFeature.leftEyePosition.y - faceWidth * 0.15, faceWidth * 0.3, faceWidth * 0.3)];
        
                leftEye.backgroundColor = [[UIColor redColor] colorWithAlphaComponent:0.4];
                leftEye.center = faceFeature.leftEyePosition;
        
                leftEye.layer.cornerRadius = faceWidth * 0.15;
                [self.view addSubview:leftEye];
            }
        
            // Boca
            if (faceFeature.hasMouthPosition) {
                // Creamos una UIView con el tamaño de la boca
                UIView *mouth = [[UIView alloc] initWithFrame:CGRectMake(faceFeature.mouthPosition.x - faceWidth * 0.20, faceFeature.mouthPosition.y - faceWidth * 0.20, faceWidth * 0.40, faceWidth * 0.40)];
        
                mouth.backgroundColor = [[UIColor greenColor] colorWithAlphaComponent:0.4];
                mouth.center = faceFeature.mouthPosition;
        
                mouth.layer.cornerRadius = faceWidth * 0.20;
                [self.view addSubview:mouth];
            }
        }
    }
    

*   El último paso sería llamar al método *faceDetector* desde *viewDidLoad* y ejecutar nuestro código para ver el resultado

<!-- -->

    - (void)viewDidLoad {
        [super viewDidLoad];
        
        [self faceDetector];
    }
    

[<img src="http://objective-c.es/wp-content/uploads/2012/11/imagen.png" alt="Imagen Chica" title="Imagen Chica" width="320" height="480" class="aligncenter size-full wp-image-306" />][2]

## Algo va mal...

Como se puede apreciar parece que algo ha salido mal y el problema está en que las coordenadas que utiliza *UIKit* y *CoreImage* (OS X) son diferentes. El origen del sistema de coordenadas de *CoreImage* están en la esquina inferior izquierda y el de *UIKit* está en la esquina superior izquierda. Por eso necesitamos invertir las posiciones antes de pintarlas por pantalla. Como la mayoría de nuestro código tendrá las coordenadas de *UIKit*, será más fácil traducir *CoreImage* a *UIKit*. Para ello tendremos que realizar una serie de modificaciones en nuestros métodos:

*faceDetector*

    -(void)faceDetector {
        // Carga la imagen de la cara
        UIImageView *imagen = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"megan.jpg"]];
        
        [self.view addSubview:imagen];
        
        // El origen del sistema de coordenadas de CoreImage están en la esquina inferior izquierda
        // y el de UIKit está en la esquina superior izquierda. Por eso necesitamos invertir las posiciones
        // antes de pintarlas por pantalla.
        CGAffineTransform transform = CGAffineTransformMakeScale(1, -1);
        transform = CGAffineTransformTranslate(transform,
                                               0,-imagen.bounds.size.height);
        
        [self markFaces:imagen withTransform:transform];
    }
    

*markFaces*: pasa a llamarse *markfaces:withTransform:* ya que vamos a pasarle un parámetro de tipo *CGAffineTransform*

    -(void)markFaces:(UIImageView *)imagenCara withTransform:(CGAffineTransform) transform {
        CIImage *imagen = [CIImage imageWithCGImage:imagenCara.image.CGImage];
        // Para el detector, utilizamos la constante "CIDetectorAccuracyHigh" la cual nos proporciona una precisión mejor pero requiere más tiempo de procesado
        // Más info en http://developer.apple.com/library/ios/#documentation/CoreImage/Reference/CIDetector_Ref/Reference/Reference.html
        CIDetector *detector = [CIDetector detectorOfType:CIDetectorTypeFace context:nil options:[NSDictionary dictionaryWithObject:CIDetectorAccuracyHigh forKey:CIDetectorAccuracy]];
        
        // Creamos un array con todas las caras detectadas en la imagen
        NSArray *features = [detector featuresInImage:imagen];
        
        // Hacemos un bucle por si detecta más de un rostro en la imagen
        for (CIFaceFeature *faceFeature in features) {
        
            // Convertir coordenadas CoreImage a UIKit
            CGRect faceRect = CGRectApplyAffineTransform(faceFeature.bounds, transform);
            // Obtener el ancho de la cara
            CGFloat faceWidth = faceFeature.bounds.size.width;
        
            // UIView usando las dimensiones de la cara detectada
            UIView *faceView = [[UIView alloc] initWithFrame:faceRect];
        
            // Añadimos un borde alrededor del UIView
            faceView.layer.borderWidth = 1;
            faceView.layer.borderColor = [[UIColor blueColor] CGColor];
        
            [self.view addSubview:faceView];
        
            // Ojo derecho
            if (faceFeature.hasRightEyePosition) {
        
                // Convertir coordenadas CoreImage a UIKit
                CGPoint rightEyePos = CGPointApplyAffineTransform(faceFeature.rightEyePosition, transform);
        
                // Creamos una UIView con el tamaño del ojo derecho
                UIView *rightEye = [[UIView alloc] initWithFrame:CGRectMake(rightEyePos.x - faceWidth * 0.15, rightEyePos.y - faceWidth * 0.15, faceWidth * 0.3, faceWidth * 0.3)];
        
                rightEye.backgroundColor = [[UIColor redColor] colorWithAlphaComponent:0.4];
                rightEye.center = rightEyePos;
        
                rightEye.layer.cornerRadius = faceWidth * 0.15;
                [self.view addSubview:rightEye];
            }
        
            // Ojo izquierdo
            if (faceFeature.hasLeftEyePosition) {
        
                // Convertir coordenadas CoreImage a UIKit
                CGPoint leftEyePos = CGPointApplyAffineTransform(faceFeature.leftEyePosition, transform);
        
                // Creamos una UIView con el tamaño del ojo izquierdo
                UIView *leftEye = [[UIView alloc] initWithFrame:CGRectMake(leftEyePos.x - faceWidth * 0.15, leftEyePos.y - faceWidth * 0.15, faceWidth * 0.3, faceWidth * 0.3)];
        
                leftEye.backgroundColor = [[UIColor redColor] colorWithAlphaComponent:0.4];
                leftEye.center = leftEyePos;
        
                leftEye.layer.cornerRadius = faceWidth * 0.15;
                [self.view addSubview:leftEye];
            }
        
            // Boca
            if (faceFeature.hasMouthPosition) {
        
                // Convertir coordenadas CoreImage a UIKit
                CGPoint mouthPos = CGPointApplyAffineTransform(faceFeature.mouthPosition, transform);
        
                // Creamos una UIView con el tamaño de la boca
                UIView *mouth = [[UIView alloc] initWithFrame:CGRectMake(mouthPos.x - faceWidth * 0.20, mouthPos.y - faceWidth * 0.20, faceWidth * 0.40, faceWidth * 0.40)];
        
                mouth.backgroundColor = [[UIColor greenColor] colorWithAlphaComponent:0.4];
                mouth.center = mouthPos;
        
                mouth.layer.cornerRadius = faceWidth * 0.20;
                [self.view addSubview:mouth];
            }
        }
    }
    

## Resultado

[<img src="http://objective-c.es/wp-content/uploads/2012/11/resultado_final.png" alt="Resultado final" title="Resultado final" width="264" height="600" class="aligncenter size-full wp-image-311" />][3]

Podéis descargar el proyecto [DetectorCaras1][4]


Fuente: [Ingens Blog][5]

 [1]: http://objective-c.es/wp-content/uploads/2012/11/Link_Binary_Libraries.jpeg
 [2]: http://objective-c.es/wp-content/uploads/2012/11/imagen.png
 [3]: http://objective-c.es/wp-content/uploads/2012/11/resultado_final.png
 [4]: http://objective-c.es/wp-content/uploads/2012/11/DetectorCaras1.zip
 [5]: http://www.ingens-networks.com/blog/post/2012/04/27/Reconocimiento-facial-en-iOS-5-con-Core-Image.aspx