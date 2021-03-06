
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Test
{
    using global::Google.Api.Ads.AdWords.Lib;
    using global::Google.Api.Ads.AdWords.v201806;
    
    using System;
    using System.Collections.Generic;

    namespace Google.Api.Ads.AdWords.Examples.CSharp.v201806
    {
        /// <summary>
        /// This code example gets keyword traffic estimates.
        /// </summary>
        public class EstimateKeywordTraffic
        {
            /// <summary>
            /// Main method, to run this code example as a standalone application.
            /// </summary>
            /// <param name="args">The command line arguments.</param>
            public static void Main(string[] args)
            {
                EstimateKeywordTraffic codeExample = new EstimateKeywordTraffic();
                Console.WriteLine(codeExample.Description);

                AdWordsUser adWordsUser = new AdWordsUser();
                AdWordsAppConfig adWordsUserConfig = (AdWordsAppConfig)adWordsUser.Config;
                adWordsUserConfig.DeveloperToken = "{token}";
                adWordsUserConfig.ClientCustomerId = "{ClientId}";
                adWordsUserConfig.OAuth2ClientId = "{clientid}";
                adWordsUserConfig.OAuth2ClientSecret = "{clientsecret}";
                adWordsUserConfig.OAuth2AccessToken = "{access token}";
                adWordsUserConfig.OAuth2RefreshToken = "{refreshtoken}";
                try
                {
                    codeExample.Run(adWordsUser);
                    
                }
                catch (Exception e)
                {
                    Console.WriteLine("An exception occurred while running this code example. {0}",
                        e.Message);
                }
                finally
                {
                    Console.ReadKey();
                }
            }

            /// <summary>
            /// Returns a description about the code example.
            /// </summary>
            public string Description
            {
                get { return "This code example gets keyword traffic estimates."; }
            }

            /// <summary>
            /// Runs the code example.
            /// </summary>
            /// <param name="user">The AdWords user.</param>
            public void Run(AdWordsUser user)
            {
                using (TrafficEstimatorService trafficEstimatorService =
                    (TrafficEstimatorService)user.GetService(AdWordsService.v201806
                        .TrafficEstimatorService))
                {
                    // Create keywords. Refer to the TrafficEstimatorService documentation for the
                    // maximum number of keywords that can be passed in a single request.
                    //   https://developers.google.com/adwords/api/docs/reference/latest/TrafficEstimatorService
                    Keyword keyword1 = new Keyword
                    {
                        text = "Seo",
                        matchType = KeywordMatchType.BROAD
                    };

                    Keyword keyword2 = new Keyword
                    {
                        text = "Seo Cong Huong",
                        matchType = KeywordMatchType.PHRASE
                    };

                    Keyword keyword3 = new Keyword
                    {
                        text = "Seo",
                        matchType = KeywordMatchType.EXACT
                    };

                    Keyword[] keywords = new Keyword[]
                    {
                    keyword1,
                    keyword2,
                    keyword3
                    };

                    // Create a keyword estimate request for each keyword.
                    List<KeywordEstimateRequest> keywordEstimateRequests =
                        new List<KeywordEstimateRequest>();

                    foreach (Keyword keyword in keywords)
                    {
                        KeywordEstimateRequest keywordEstimateRequest = new KeywordEstimateRequest
                        {
                            keyword = keyword,
                            
                        };
                        keywordEstimateRequests.Add(keywordEstimateRequest);
                    }

                    

                    // Create ad group estimate requests.
                    AdGroupEstimateRequest adGroupEstimateRequest = new AdGroupEstimateRequest
                    {
                        keywordEstimateRequests = keywordEstimateRequests.ToArray(),
                        maxCpc = new Money
                        {
                            microAmount = 1000000
                        },
                    };

                    // Create campaign estimate requests.
                    CampaignEstimateRequest campaignEstimateRequest = new CampaignEstimateRequest
                    {
                        adGroupEstimateRequests = new AdGroupEstimateRequest[]
                        {
                        adGroupEstimateRequest
                        },
                        networkSetting = new NetworkSetting()
                        {
                            targetGoogleSearch = true
                        }
                    };

                    // Optional: Set additional criteria for filtering estimates.
                    // See http://code.google.com/apis/adwords/docs/appendix/countrycodes.html
                    // for a detailed list of country codes.
                    Location countryCriterion = new Location
                    {
                        id = 1028581 //Ho Chi Minh
                    };

                    // See http://code.google.com/apis/adwords/docs/appendix/languagecodes.html
                    // for a detailed list of language codes.
                    Language languageCriterion = new Language
                    {
                        id = 1040
                        //vi
                    };

                    campaignEstimateRequest.criteria = new Criterion[]
                    {
                    countryCriterion,
                    languageCriterion
                    };

                    try
                    {
                        // Create the selector.
                        TrafficEstimatorSelector selector = new TrafficEstimatorSelector()
                        {
                            campaignEstimateRequests = new CampaignEstimateRequest[]
                            {
                            campaignEstimateRequest
                            },

                            // Optional: Request a list of campaign level estimates segmented by
                            // platform.
                            platformEstimateRequested = true
                        };

                        // Get traffic estimates.
                        TrafficEstimatorResult result = trafficEstimatorService.get(selector);

                        // Display traffic estimates.
                        if (result != null && result.campaignEstimates != null &&
                            result.campaignEstimates.Length > 0)
                        {
                            CampaignEstimate campaignEstimate = result.campaignEstimates[0];

                            // Display the campaign level estimates segmented by platform.
                            if (campaignEstimate.platformEstimates != null)
                            {
                                foreach (PlatformCampaignEstimate platformEstimate in campaignEstimate
                                    .platformEstimates)
                                {
                                    string platformMessage = string.Format(
                                        "Results for the platform with ID: " + "{0} and name : {1}.",
                                        platformEstimate.platform.id,
                                        platformEstimate.platform.platformName);

                                    DisplayMeanEstimates(platformMessage, platformEstimate.minEstimate,
                                        platformEstimate.maxEstimate);
                                }
                            }

                            // Display the keyword estimates.
                            if (campaignEstimate.adGroupEstimates != null &&
                                campaignEstimate.adGroupEstimates.Length > 0)
                            {
                                AdGroupEstimate adGroupEstimate = campaignEstimate.adGroupEstimates[0];

                                if (adGroupEstimate.keywordEstimates != null)
                                {
                                    for (int i = 0; i < adGroupEstimate.keywordEstimates.Length; i++)
                                    {
                                        Keyword keyword = keywordEstimateRequests[i].keyword;
                                        KeywordEstimate keywordEstimate =
                                            adGroupEstimate.keywordEstimates[i];

                                        if (keywordEstimateRequests[i].isNegative)
                                        {
                                            continue;
                                        }

                                        string kwdMessage = string.Format(
                                            "Results for the keyword with text = '{0}' " +
                                            "and match type = '{1}':", keyword.text, keyword.matchType);
                                        DisplayMeanEstimates(kwdMessage, keywordEstimate.min,
                                            keywordEstimate.max);
                                    }
                                }
                            }
                        }
                        else
                        {
                            Console.WriteLine("No traffic estimates were returned.");
                        }

                        trafficEstimatorService.Close();
                    }
                    catch (Exception e)
                    {
                        throw new System.ApplicationException("Failed to retrieve traffic estimates.",
                            e);
                    }
                }
            }

            /// <summary>
            /// Displays the mean estimates.
            /// </summary>
            /// <param name="message">The message to display.</param>
            /// <param name="minEstimate">The minimum stats estimate.</param>
            /// <param name="maxEstimate">The maximum stats estimate.</param>
            private void DisplayMeanEstimates(string message, StatsEstimate minEstimate,
                StatsEstimate maxEstimate)
            {
                // Find the mean of the min and max values.
                long meanAverageCpc = 0;
                double meanAveragePosition = 0;
                float meanClicks = 0;
                long meanTotalCost = 0;

                if (minEstimate != null && maxEstimate != null)
                {
                    if (minEstimate.averageCpc != null && maxEstimate.averageCpc != null)
                    {
                        meanAverageCpc = (minEstimate.averageCpc.microAmount +
                            maxEstimate.averageCpc.microAmount) / 2;
                    }

                    if (minEstimate.averagePositionSpecified && maxEstimate.averagePositionSpecified)
                    {
                        meanAveragePosition =
                            (minEstimate.averagePosition + maxEstimate.averagePosition) / 2;
                    }

                    if (minEstimate.clicksPerDaySpecified && maxEstimate.clicksPerDaySpecified)
                    {
                        meanClicks = (minEstimate.clicksPerDay + maxEstimate.clicksPerDay) / 2;
                    }

                    if (minEstimate.totalCost != null && maxEstimate.totalCost != null)
                    {
                        meanTotalCost = (minEstimate.totalCost.microAmount +
                            maxEstimate.totalCost.microAmount) / 2;
                    }
                }

                Console.WriteLine(message);
                Console.WriteLine("  Estimated average CPC: {0}", meanAverageCpc);   //always 0
                Console.WriteLine("  Estimated ad position: {0:0.00}", meanAveragePosition);  //always 0
                Console.WriteLine("  Estimated daily clicks: {0}", meanClicks);  //always 0
                Console.WriteLine("  Estimated daily cost: {0}", meanTotalCost);  //always 0
            }
        }
    }
}




