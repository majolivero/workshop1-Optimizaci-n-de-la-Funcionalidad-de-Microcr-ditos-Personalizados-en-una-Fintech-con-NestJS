# workshop1-Optimizacion-de-la-Funcionalidad-de-Microcreditos-Personalizados-en-una-Fintech-con-NestJS

# 🏷️ Optimización de la Funcionalidad de Microcréditos Personalizados en una Fintech con NestJS

---

### 🎯 **Objetivo del Taller:**  
Los estudiantes aprenderán a aplicar los principios SOLID en un proyecto realista de una fintech, optimizando la calidad del código y mejorando su mantenibilidad.

---

### Introducción a SOLID

**¿Qué es SOLID?**

SOLID es un acrónimo que representa cinco principios fundamentales de la programación orientada a objetos que, cuando se aplican correctamente, ayudan a crear sistemas de software más robustos, flexibles y mantenibles. Estos principios fueron introducidos por Robert C. Martin y son considerados un estándar en el desarrollo de software.

**Principios SOLID:**

1. **S**ingle Responsibility Principle (SRP) - Principio de Responsabilidad Única
2. **O**pen/Closed Principle (OCP) - Principio de Abierto/Cerrado
3. **L**iskov Substitution Principle (LSP) - Principio de Sustitución de Liskov
4. **I**nterface Segregation Principle (ISP) - Principio de Segregación de Interfaces
5. **D**ependency Inversion Principle (DIP) - Principio de Inversión de Dependencias

Estos principios buscan mejorar la estructura del código, facilitando su mantenimiento, extensión y testeo. A continuación, veremos cómo aplicar cada uno de estos principios en la implementación de una funcionalidad de microcréditos en una fintech usando NestJS, y cómo sería el código si no se aplicaran estos principios.

---

### 📜 **Historia de Usuario:**  
"Como desarrollador de una fintech, quiero implementar una mejora en la funcionalidad de microcréditos que permita a los usuarios recibir ofertas personalizadas basadas en su historial de crédito y comportamiento financiero, de modo que puedan acceder a mejores condiciones de crédito de manera eficiente."

### **Entidades Clave**
- **User:** Representa a los usuarios de la plataforma.  
  **Atributos:**
  - `id: string`
  - `name: string`
  - `creditScore: number`
  - `financialHistory: FinancialRecord[]` (array de registros financieros)
  
- **Microcredit:** Representa una solicitud de microcrédito.  
  **Atributos:**
  - `id: string`
  - `userId: string`
  - `amount: number`
  - `interestRate: number`
  - `status: string`

---

## **Desglose del Taller con Código en NestJS**

### 1. **Principio S (Single Responsibility Principle) - Principio de Responsabilidad Única**

**¿Qué es?**

El principio de responsabilidad única establece que una clase debería tener una única responsabilidad o razón para cambiar. Esto significa que cada clase debe tener una única tarea o responsabilidad dentro del sistema.

#### Sin SRP (No se debería hacer):

```typescript
@Injectable()  //Decorador que indica que esta clase puede ser inyectada como una dependencia en otras clases de NestJS. 
export class MicrocreditService {  //Define la clase MicrocreditService, que es donde se implementan las funcionalidades relacionadas con microcréditos 
  constructor(private readonly userRepository: UserRepository) {} //Define un constructor que inyecta la dependencia UserRepository, la cual, se utiliza para acceder a la información del usuario en la base de datos. 

  applyForMicrocredit(userId: string, amount: number): Microcredit {  //Método que se encarga de procesar la solicitud de un microcrédito para un usuario dado. 
    const user = this.userRepository.findById(userId); //Obtiene el usuario de la base de datos utilizando su ID.

    // Lógica para calcular la tasa de interés
    const interestRate = this.calculateInterestRate(user); //Llama al método calculateInterestRate para calcular la tasa de interés basada en el historial de crédito del usuario. 

    // Lógica para registrar el microcrédito
    const microcredit = new Microcredit(userId, amount, interestRate, 'PENDING');  //Crea una nueva instancia de Microcredit con los datos proporcionados. 
    this.saveMicrocredit(microcredit);

    return microcredit;  //Retorna el microcrédito creado
  }

  private calculateInterestRate(user: User): number {   //Método privado que calcula la tasa de interés basada en el historial de crédito del usuario. 
    // Lógica de cálculo de tasa de interés basada en el historial de crédito
    return user.creditScore > 700 ? 5 : 15;  //Devuelve la tasa de interés basada en la puntuación del usuario (5% si el puntaje es mayor a 700, en caso contrario, el 15%.)
  }

  private saveMicrocredit(microcredit: Microcredit): void {  //Método privado que guarda el microcrédito en la base de datos. 
    // Lógica para guardar el microcrédito en la base de datos
  }
}
```
**ANÁLISIS DE POR QUE NO SE CUMPLE EL SRP EN EL CÓDIGO ANTERIOR Y CONSECUENCIAS**

Este enfoque no respeta el principio de responsabilidad única ya que la clase MicrocreditService está realizando múltiples tareas: 

1. Interactuar con el repositorio de usuarios para obtener datos del usuario.
2. Calcular la tasa de interés para el microcrédito.
3. Registrar el microcrédito en la base de datos. 

Estas son tres responsabilidades distintas que podrían cambiar por diferentes motivos(cambio en la lógica de cáculo de tasas, cambio en como se guarda el microcrédito,etc.)
Cualquier cambio en una de estas responsabilidades podría tener varias consecuencias negativas:

1. Si la lógica de cálculo de la tasa de interés necesita cambiar, se tendría que modificar el método calculateInterestRate dentro de la clase MicrocreditService. Esto puede implicar una prueba exhaustiva de todo el servicio, ya que todos los métodos están en la misma clase. También hace que la clase sea más difícil de entender y mantener, ya que combina diferentes lógicas en un solo lugar.

2. Si la forma de guardar el microcrédito en la base de datos cambia(por ejemplo, si se decide usar un ORM diferente o se cambia la estructura de la base de datos), se tendría que modificar el método saveMicrocredit. Esto aumenta el riesgo de introducir errores en el código que no están directamente relacionados con el cambio realizado. 

PRUEBAS UNITARIAS:
Al no cumplir este principio, las pruebas unitarias se vuelven más complicadas. Cada prueba debe cubrir no sola la lógica de solicitud de microcrédito, sino también la lógica de cálculo de tasas y la lógica de almacenamiento. Esto puede llevar a pruebas más largas y dificiles de mantener. 

MENOR REUTILIZACIÓN DEL CÓDIGO:
Las funciones calculateInterestRate y saveMicrocredit estan directamente ligadas a MicrocreditService. Si necesitas calcular la tasa de interés o guardar un microcrédito en otro contexto, se tendría que duplicar el código o acceder a MicrocreditService, lo cual no es ideal. Esto va en contra del principio DRY.(Don't Repeat Yourself).
Si cada responsabilidad estuviera en su propio servicio, se podrían reutilizar esos servicios en otros contextos sin duplicar la lógica. 

DIFICULTAR PARA ESCALAR O MODIFICAR FUNCIONALIDADES:
Si la aplicación crece y se agregan nuevas características o reglas de negocio, tener múltiples responsabilidades en una sola clase puede hacer que sea dificil escalar el código.
Si MicrocreditService necesita ser extendido o modificado, la mezcla de responsabilidades puede hacer que esas modificaciones sean más complejas y propensas a errores. 


#### Con SRP (Así se debería hacer):***

```typescript
@Injectable()
export class CreditCalculationService {  //Se encarga únicamente del cálculo de la tasa de interés
  calculateInterestRate(user: User): number {
    return user.creditScore > 700 ? 5 : 15;
  }
}

@Injectable()
export class MicrocreditRegistryService {  //Se encarga de guardar el microcrédito en la base de datos
  saveMicrocredit(microcredit: Microcredit): void {  
    // Lógica para guardar el microcrédito en la base de datos
  }
}

//Servicio Principal que coordina las operaciones 
@Injectable()
export class MicrocreditService {   //Clase responsable de coordinar el proceso de solicitud de microcrédito. 
  constructor(   //El constructor recibe tres dependencias inyectadas. 
    private readonly userRepository: UserRepository,  //Permite obtener información de los usuarios desde un repositorio de datos. 
    private readonly creditCalculationService: CreditCalculationService,  //Se utiliza para calcular la tasa de interés. 
    private readonly microcreditRegistryService: MicrocreditRegistryService //Se utiliza para guardar el microcrédito. 
  ) {}

  applyForMicrocredit(userId: string, amount: number): Microcredit {  //Método principal que gestiona la solicitud de un microcrédito. Devuelve un objeto Microcredit. 
    const user = this.userRepository.findById(userId);  //Obtiene el objeto User asociado al userId proporcionado utilizando el userRepository
    const interestRate = this.creditCalculationService.calculateInterestRate(user);  //Utiliza el servicio CreditCalculationService para calcular la tasa de interés del usuario. 
    const microcredit = new Microcredit(userId, amount, interestRate, 'PENDING');  //Crea una nueva instancia de Microcredit con el userId, amount, interestRate, y un estado inicial de PENDING. 
    this.microcreditRegistryService.saveMicrocredit(microcredit); //Utiliza el servicio microcreditRegistryService para guardar el microcrédito en la base de datos. 
    return microcredit; //Retorna el microcrédito que fue creado y guardado. 
  }
}

```

### 2. **Principio O (Open/Closed Principle) - Principio de Abierto/Cerrado**

**¿Qué es?**

El principio de Abierto/Cerrado establece que una clase debe estar abierta a la extensión pero cerrada a la modificación. Esto significa que deberíamos poder añadir nuevas funcionalidades sin alterar el código existente.

#### Sin OCP (No se debería hacer):

```typescript
@Injectable()
export class CreditCalculationService {
  calculateInterestRate(user: User): number {
    if (user.creditScore > 700) {
      return 5;
    } else if (user.creditScore > 500) {
      return 10;
    } else {
      return 15;
    }
  }
}
```

Este código viola el principio OCP porque cada vez que necesitamos modificar la lógica de cálculo de interés, debemos alterar el código existente.

#### Con OCP (Así se debería hacer):

```typescript
interface InterestRateStrategy {
  calculate(user: User): number;
}

@Injectable()
export class StandardInterestRateStrategy implements InterestRateStrategy {
  calculate(user: User): number {
    return user.creditScore > 700 ? 5 : 15;
  }
}

@Injectable()
export class PremiumInterestRateStrategy implements InterestRateStrategy {
  calculate(user: User): number {
    return user.creditScore > 700 ? 3 : 10;
  }
}

@Injectable()
export class CreditCalculationService {
  private strategy: InterestRateStrategy;

  constructor(strategy: InterestRateStrategy) {
    this.strategy = strategy;
  }

  calculateInterestRate(user: User): number {
    return this.strategy.calculate(user);
  }
}
```

Al utilizar una estrategia de cálculo de interés, podemos añadir nuevas estrategias sin modificar el código existente, haciendo que la clase esté abierta para la extensión pero cerrada a la modificación.

### 3. Principio L (Liskov Substitution Principle) - Principio de Sustitución de Liskov

***¿Qué es?***

El principio de Sustitución de Liskov establece que los objetos de una clase base deben poder ser reemplazados por objetos de una clase derivada sin alterar la funcionalidad del programa.

#### Sin LSP (No se debería hacer):

```typescript
class BasicMicrocredit extends Microcredit {
  constructor(userId: string, amount: number) {
    super(userId, amount, 'PENDING');
  }

  // Sobrescribe un método de la clase base, pero no funciona con todas las subclases
  override apply(): void {
    // Lógica específica para microcréditos básicos
  }
}

class AdvancedMicrocredit extends Microcredit {
  constructor(userId: string, amount: number) {
    super(userId, amount, 'APPROVED');
  }

  override apply(): void {
    // Lógica específica para microcréditos avanzados
  }
}
```

Este código rompe el principio de Liskov porque las subclases modifican el comportamiento de la clase base de manera que podría no ser esperado por otras partes del código.

#### Con LSP (Así se debería hacer):

```typescript
class Microcredit {
  apply(): void {
    // Lógica genérica para aplicar un microcrédito
  }
}

class BasicMicrocredit extends Microcredit {
  // No es necesario sobrescribir el método apply si no altera el comportamiento
}

class AdvancedMicrocredit extends Microcredit {
  // Aquí podrías extender el comportamiento si es necesario, pero sin romper el contrato
}
```

Las subclases deben respetar el comportamiento de la clase base, garantizando que cualquier clase que sustituya a la clase base mantendrá el comportamiento esperado.