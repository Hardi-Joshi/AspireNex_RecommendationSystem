# AspireNex_RecommendationSystem
```Java
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;
class UserProfile {
    private String name;
    private Map<String, Integer> ratings;

    public UserProfile(String name) {
        this.name = name;
        this.ratings = new HashMap<>();
    }
    public String getName() {
        return name;
    }

    public void addRating(String item, int rating) {
        ratings.put(item, rating);
    }

    public Map<String, Integer> getRatings() {
        return ratings;
    }
}
class ItemAttributes {
    Map<String, Map<String, Double>> itemAttributes;

    public ItemAttributes() {
        itemAttributes = new HashMap<>();
    }

    public void addAttributes(String item, Map<String, Double> attributes) {
        itemAttributes.put(item, attributes);
    }

    public Map<String, Double> getAttributes(String item) {
        return itemAttributes.getOrDefault(item, new HashMap<>());
    }
}
class UserPreferences {
    Map<String, UserProfile> userProfiles;

    public UserPreferences() {
        userProfiles = new HashMap<>();
    }

    public void addUserProfile(UserProfile userProfile) {
        userProfiles.put(userProfile.getName(), userProfile);
    }

    public UserProfile getUserProfile(String user) {
        return userProfiles.get(user);
    }
}
class Recommender {
    UserPreferences userPreferences;
    ItemAttributes itemAttributes;

    public Recommender(UserPreferences userPreferences, ItemAttributes itemAttributes) {
        this.userPreferences = userPreferences;
        this.itemAttributes = itemAttributes;
    }
    public Map<String, Double> recommend(String user) {
        Map<String, Double> recommendations = new HashMap<>();
        UserProfile userProfile = userPreferences.getUserProfile(user);
        if(userProfile== null) return recommendations;
        Map<String, Integer> userRatings = userProfile.getRatings();

        for (UserProfile disUserProfile : userPreferences.userProfiles.values()){
            if (!disUserProfile.getName().equals(user)) {
                Map<String, Integer> disUserRatings = disUserProfile.getRatings();
                for (Map.Entry<String, Integer> itemRating : disUserRatings.entrySet()) {
                    String item = itemRating.getKey();
                    int rating = itemRating.getValue();
                    if (!userRatings.containsKey(item)) {
                        recommendations.put(item, recommendations.getOrDefault(item, 0.0) + rating);
                    }
                }
            }
        }
        for (String item : itemAttributes.itemAttributes.keySet()) {
            if (!userRatings.containsKey(item)) {
                double score = ContentSimilarity(userRatings, item);
                recommendations.put(item, recommendations.getOrDefault(item, 0.0) + score);
            }
        }
        return recommendations;
    }
private double ContentSimilarity(Map<String, Integer> userRatings, String item) {
        double similarity = 0.0;
        Map<String, Double> targetItemAttributes = itemAttributes.getAttributes(item);

        for (Map.Entry<String, Integer> entry : userRatings.entrySet()) {
            String ratedItem = entry.getKey();
            int rating = entry.getValue();
            Map<String, Double> ratedItemAttributes = itemAttributes.getAttributes(ratedItem);
            double dotProduct = 0.0;
            double normTarget = 0.0;
            double normRated = 0.0;

            for (String attribute : targetItemAttributes.keySet()) {
                double targetValue = targetItemAttributes.getOrDefault(attribute, 0.0);
                double ratedValue = ratedItemAttributes.getOrDefault(attribute, 0.0);
                dotProduct += targetValue * ratedValue;
                normTarget += targetValue * targetValue;
                normRated += ratedValue * ratedValue;
            }
            if (normTarget > 0 && normRated > 0) {
                similarity += rating * (dotProduct / (Math.sqrt(normTarget) * Math.sqrt(normRated)));
            }
        }

        return similarity;
    }
}
public class RecommendationSystem{
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        UserPreferences preferences = new UserPreferences();
        ItemAttributes attributes = new ItemAttributes();

        Map<String, Double> movie1Attributes = new HashMap<>();
        movie1Attributes.put("Action", 0.8);
        movie1Attributes.put("Comedy", 0.2);
        attributes.addAttributes("Movie1", movie1Attributes);

        Map<String, Double> movie2Attributes = new HashMap<>();
        movie2Attributes.put("Action", 0.4);
        movie2Attributes.put("Comedy", 0.6);
        attributes.addAttributes("Movie2", movie2Attributes);

        Map<String, Double> movie3Attributes = new HashMap<>();
        movie3Attributes.put("Action", 0.9);
        movie3Attributes.put("Comedy", 0.1);
        attributes.addAttributes("Movie3", movie3Attributes);

        System.out.println("Enter the number of users:");
        int numUsers = scanner.nextInt();
        scanner.nextLine(); 

        for(int i = 0; i < numUsers; i++){
            System.out.println("Enter user name:");
            String userName = scanner.nextLine();
            UserProfile userProfile = new UserProfile(userName);

            System.out.println("Enter the number of ratings for " + userName + ":");
            int numRatings = scanner.nextInt();
            scanner.nextLine(); 

            for (int j = 0; j < numRatings; j++) {
                System.out.println("Enter item name:");
                String itemName = scanner.nextLine();
                int rating;
                while (true) {
                    System.out.println("Enter rating for " + itemName + " (1-5):");
                    rating = scanner.nextInt();
                    scanner.nextLine();
                    if (rating >= 1 && rating <= 5) {
                        break;
                    } else {
                        System.out.println("Invalid rating. Please enter a rating between 1 and 5.");
                    }
                }
                userProfile.addRating(itemName, rating);
            }

            preferences.addUserProfile(userProfile);
        }
Recommender recommender = new Recommender(preferences, attributes);

        System.out.println("Enter user name to get recommendations:");
        String userForRecommendation = scanner.nextLine();

        Map<String, Double> recommendations = recommender.recommend(userForRecommendation);

        System.out.println("Recommendations for " + userForRecommendation + ":");
        for (Map.Entry<String, Double> entry : recommendations.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }

        scanner.close();
    }
}
```
