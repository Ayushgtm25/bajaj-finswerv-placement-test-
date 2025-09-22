# bajaj-finswerv-placement-test-
spring boot application 
package com.bajajfinserv.service;

import com.bajajfinserv.model.SqlSubmissionRequest; 
import com.bajajfinserv.model.WebhookRequest;
import com.bajajfinserv.model.WebhookResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

@Service
public class PlacementTestService {
    
    private static final Logger logger = LoggerFactory.getLogger(PlacementTestService.class);
    
    private final WebClient webClient;
    
    @Value("${app.student.name}")
    private String studentName;
    
    @Value("${app.student.regNo}")
    private String registrationNumber;
    
    @Value("${app.student.email}")
    private String studentEmail;
    
    @Value("${app.api.base-url}")
    private String apiBaseUrl;
    
    public PlacementTestService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.build();
    }
    
    /**
     * Main method to execute the complete placement test process
     */
    public void executeCompleteProcess() {
        try {
            logger.info("Starting Bajaj Finserv Health placement test automation...");
            
            // Step 1: Generate webhook and get access token
            WebhookResponse webhookResponse = generateWebhook();
            
            if (webhookResponse == null) {
                logger.error("Failed to generate webhook. Stopping process.");
                return;
            }
            
            logger.info("Successfully received webhook URL and access token");
            
            // Step 2: Solve SQL problem based on registration number
            String sqlQuery = solveSqlProblem();
            
            // Step 3: Submit the solution
            submitSolution(webhookResponse.getWebhookUrl(), 
                          webhookResponse.getAccessToken(), 
                          sqlQuery);
            
            logger.info("Placement test automation completed successfully!");
            
        } catch (Exception e) {
            logger.error("Error during placement test execution: ", e);
        }
    }
    
    /**
     * Step 1: Send POST request to generate webhook
     */
    public WebhookResponse generateWebhook() {
        try {
            logger.info("Sending webhook generation request...");
            
            WebhookRequest request = new WebhookRequest(studentName, registrationNumber, studentEmail);
            
            WebhookResponse response = webClient
                    .post()
                    .uri(apiBaseUrl)
                    .contentType(MediaType.APPLICATION_JSON)
                    .bodyValue(request)
                    .retrieve()
                    .bodyToMono(WebhookResponse.class)
                    .block();
            
            logger.info("Webhook generation response: {}", response);
            return response;
            
        } catch (Exception e) {
            logger.error("Error generating webhook: ", e);
            return null;
        }
    }
    
    /**
     * Step 2: Solve SQL problem based on registration number
     */
    public String solveSqlProblem() {
        // Extract last two digits from registration number
        String lastTwoDigits = registrationNumber.substring(registrationNumber.length() - 2);
        int lastTwoDigitsInt = Integer.parseInt(lastTwoDigits);
        
        logger.info("Last two digits of registration number: {}", lastTwoDigits);
        
        String sqlQuery;
        
        if (lastTwoDigitsInt % 2 == 1) { // Odd number - Question 1
            logger.info("Solving Question 1 (odd registration number ending)");
            sqlQuery = solveQuestion1();
        } else { // Even number - Question 2
            logger.info("Solving Question 2 (even registration number ending)");
            sqlQuery = solveQuestion2();
        }
        
        logger.info("Generated SQL Query: {}", sqlQuery);
        return sqlQuery;
    }
    
    /**
     * Solve Question 1 (for odd last two digits)
     */
    private String solveQuestion1() {
        // Example SQL query for Question 1
        // Replace this with the actual SQL query for Question 1
        return """
            SELECT 
                customer_id,
                customer_name,
                COUNT(order_id) as total_orders,
                SUM(order_amount) as total_amount
            FROM 
                customers c
            LEFT JOIN 
                orders o ON c.customer_id = o.customer_id
            WHERE 
                c.registration_date >= '2023-01-01'
            GROUP BY 
                c.customer_id, c.customer_name
            HAVING 
                COUNT(order_id) > 5
            ORDER BY 
                total_amount DESC;
            """;
    }
    
    /**
     * Solve Question 2 (for even last two digits)
     */
    private String solveQuestion2() {
        // Example SQL query for Question 2
        // Replace this with the actual SQL query for Question 2
        return """
            SELECT 
                product_category,
                AVG(product_price) as average_price,
                MAX(product_price) as max_price,
                MIN(product_price) as min_price
            FROM 
                products p
            JOIN 
                categories c ON p.category_id = c.category_id
            WHERE 
                p.is_active = true
                AND p.stock_quantity > 0
            GROUP BY 
                product_category
            ORDER BY 
                average_price DESC;
            """;
    }
    
    /**
     * Step 3: Submit the solution to webhook URL
     */
    public void submitSolution(String webhookUrl, String accessToken, String sqlQuery) {
        try {
            logger.info("Submitting solution to webhook URL: {}", webhookUrl);
            
            SqlSubmissionRequest submissionRequest = new SqlSubmissionRequest(sqlQuery);
            
            String response = webClient
                    .post()
                    .uri(webhookUrl)
                    .header(HttpHeaders.AUTHORIZATION, "Bearer " + accessToken)
                    .header(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                    .bodyValue(submissionRequest)
                    .retrieve()
                    .bodyToMono(String.class)
                    .block();
            
            logger.info("Solution submission response: {}", response);
            
        } catch (Exception e) {
            logger.error("Error submitting solution: ", e);
        }
    }
}
