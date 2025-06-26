//driver
package task1;

import java.util.Random;
import java.util.Scanner;

public class Driver {


    private static final Random randomizer = new Random();
    private static double previousSensorValue = -1;

    public static void main(String[] args) {
        Scanner inputScanner = new Scanner(System.in);
        File_Logger activityLogger = new File_Logger("log.txt");


        System.out.print("Set a max temperature threshold (minimum 20°C): ");
        double tempLimit = inputScanner.nextDouble();
        if (tempLimit < 20) {
            System.out.println("Input below minimum. Defaulting to 20°C.");
            tempLimit = 20;
        }


        System.out.print("Choose humidity mode — type 1 for random or 0 for manual entry: ");
        int humidityInputType = inputScanner.nextInt();
      
        for (int readingNum = 1; readingNum <= 10; readingNum++) {
            System.out.println("Measurement #" + readingNum);

            double temperature = simulateTemperature(tempLimit);
            double humidity;

            if (humidityInputType == 1) {
                humidity = simulateHumidity();
            } else {
                System.out.print("Manually enter humidity value (0.0 - 1.0): ");
                humidity = inputScanner.nextDouble();
                if (humidity < 0 || humidity > 1) {
                    System.out.println("Invalid input. Defaulting to 0.5.");
                    humidity = 0.5;
                }
            }

            System.out.printf("Temp: %.2f°C\n", temperature);
            System.out.printf("Humidity (scaled): %.2f\n", humidity);

            double sensor1 = sensor3Reading();
            double sensor2 = sensor3Reading();
            double sensor3 = sensor3Reading();

            System.out.printf("Sensor3 samples: [%.2f, %.2f, %.2f]\n", sensor1, sensor2, sensor3);

            double consensus = evaluateMajority(sensor1, sensor2, sensor3);

            if (consensus == -1) {

                if (previousSensorValue == -1) {
                    previousSensorValue = (sensor1 + sensor2 + sensor3) / 3.0;
                    System.out.println("No prior reference. Calculated average used.");
                } else {
                    System.out.println("Disagreement in values. Falling back to previous stable result.");
                }

                String discrepancyLog = String.format(
                    "MISMATCH: Sensor3 readings = [%.2f, %.2f, %.2f]. Fallback used: %.2f",
                    sensor1, sensor2, sensor3, previousSensorValue
                );
                activityLogger.log(discrepancyLog);
                consensus = previousSensorValue;
            } else {

                previousSensorValue = consensus;
            }

            System.out.printf("Resolved Sensor3 output: %.2f\n", consensus);
            
        }

        inputScanner.close();
    }


    public static double simulateTemperature(double maxCap) {
        return 20 + randomizer.nextDouble() * (maxCap - 20);
    }


    public static double simulateHumidity() {
        return randomizer.nextDouble();
    }


    public static double sensor3Reading() {
        return 100 + randomizer.nextInt(6);
    }

 
    public static double evaluateMajority(double val1, double val2, double val3) {
        final double margin = 0.01;

        boolean firstSecond = Math.abs(val1 - val2) < margin;
        boolean firstThird = Math.abs(val1 - val3) < margin;
        boolean secondThird = Math.abs(val2 - val3) < margin;

        if (firstSecond || firstThird) return val1;
        if (secondThird) return val2;

        return -1;
    }
}
